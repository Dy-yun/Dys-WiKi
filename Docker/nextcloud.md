# NextCloud

> 分别采用doker-compose和docker-ce两种方式搭建NextCloud，数据库均使用mysql:laster。

2021/09/06 

---

# docker-compose

> nextcloud官方文档运行后mariadb会出现写入错误，貌似是新版nextcloud的数据库命令冲突，所以改用了mysql。mysql:5.7也会出现同样的错误，但数据库可以创建成功，就是一直在报错，有点迷。

- 新建一个文件夹，这里新建了一个/root/nextcloud,再分别新建nextcloud和db作为程序根目录和数据库根目录

  ```bash
  mkdir /root/nextcloud
  mkdir /root/nextcloud/nextcloud
  mkdir /root/nextcloud/db
  ```

- 在/root/nextcloud下新建一个docker-compose.yml文件

  ```bash
  vim docker-compose.yml
  ```

- 输入以下内容:wq保存退出，记得填好自己的数据库密码和挂载地址

  ```yaml
  version: "2"

  services:
    nextcloud-db:
      image: mysql
      restart: always
      volumes:
        - /root/nextcloud/db:/var/lib/mysql
      environment:
        - MYSQL_ROOT_PASSWORD=`数据库ROOT密码`
        - MYSQL_PASSWORD=`数据库密码`
        - MYSQL_DATABASE=nextcloud
        - MYSQL_USER=nextcloud
  
    nextcloud:
      image: nextcloud
      restart: always
      ports:
        - 9880:80
      volumes:
        - /root/nextcloud/nextcloud:/var/www/html
        - /root/nextcloud/data:/var/www/html/data
      environment:
        - MYSQL_PASSWORD=`数据库密码`
        - MYSQL_DATABASE=nextcloud
        - MYSQL_USER=nextcloud
        - MYSQL_HOST=nextcloud-db:3306
  		  - UID=1000
  	    - GID=1000
  	    - UPLOAD_MAX_SIZE=10G
  	    - APC_SHM_SIZE=128M
  	    - OPCACHE_MEM_SIZE=128
  	    - CRON_PERIOD=15m
  	    - TZ=Aisa/Shanghai
  	    - ADMIN_USER=`管理员账户`
  	    - ADMIN_PASSWORD=`管理员密码`

  ```

> 如果data是挂载的可能会出现没有写入权限，这时候需要加上这句提权

  ```yaml
  privileged: true
  ```

> 提权后登入可能会显示设置为0770权限，启动起来后要进去容器/config/config.php加上这一句

  ```php
  'check_data_directory_permissions' => false
  ```

- 启动docker-compose.yml,-d后台运行

  ```bash
  docker-compose up -d
  ```

> 启动完成后，输入你的ip:9880进入nextcloud页面配置，下方有图，数据库配置没问题的情况下，只需要创建管理员账户，等待即可完成。
       部分配置较低的机器（比如我的1C2G）可能会卡住，怎么刷新都卡在了创建用户界面，这时只需要把容器dowm再up就可以了，配置会留在映射的宿主机地址，如果需要完全重新创建，记得去宿主机删除对应的映射文件夹或者换一个地址。

# docker-ce + network

> 容器间采用network链接网络，当然也可以使用--link参数。使用docker-compose是自动创建在同一个网络下。

- 拉取镜像

  ```bash
  #nextcloud
  docker pull nextcloud
  #mysql
  docker pull mysql
  #mysql可视化管理工具
  docker pull phpmyadmin
  ```

- 查看镜像列表

  ```bash
  docker images
  ```

- 创建网络

  ```bash
  docker network create network-nextcloud
  ```

- 启动nextwork容器

  ```bash
  docker run -d  \
  --network network-nextcloud \
  --network-alias nextcloud \
  --restart=always \
  --name nextcloud \
  -v nextcloud:/var/www/html \
  -p 888:80 \
  nextcloud
  ```

- 启动mysql容器

  ```bash
  docker run -d \
  --network network-nextcloud \
  --network-alias mysql \
  --restart=always \
  --name nextcloud_db \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -p 3306:3306 \
  mysql
  ```

- 启动myadmin容器

  ```bash
  docker run -d \
  --network network-nextcloud \
  --network-alias myadmin \
  --restart=always \
  --name nextcloud_db_myadmin \
  -e PMA_HOST=mysql \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -e PMA_PORT=3306 \
  -p 8088:80 \
  phpmyadmin
  ```

- ip:888进入nextcloud主页，本地ip为localhost
配置管理员账户密码，数据库选择mysql，数据库主机输入mysql:3306，当然也可以通过环境变量全部预设好，参数参考上面docker-compose.yml

  ![606AE42C-C8AD-46C4-AADC-3AFCAC26D01D.png](https://img.dycloud.work/images/2021/09/13/606AE42C-C8AD-46C4-AADC-3AFCAC26D01D.png)

- 完成

  ![EA1BC3A3-D6A9-400C-BF4C-B22C33387A75.png](https://img.dycloud.work/images/2021/09/13/EA1BC3A3-D6A9-400C-BF4C-B22C33387A75.png)
 