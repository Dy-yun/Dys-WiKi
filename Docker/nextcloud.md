# NextCloud

> 分别采用doker-compose和docker-ce两种方式搭建NextCloud，数据库均使用mariadb。

2021/09/06 

---

<!-- tabs:start -->
#### **docker-compose**

> nextcloud官方文档运行后mariadb会出现写入错误，貌似是新版nextcloud的数据库命令冲突，所以改用了mysql。mysql:5.7也会出现同样的错误，但数据库可以创建成功，就是一直在报错，有点迷。

- 新建一个文件夹，这里新建了一个/root/nextcloud作为根目录

  ```bash
  mkdir /root/nextcloud
  ```

- 在/root/nextcloud下新建一个docker-compose.yml文件

  ```bash
  vim docker-compose.yml
  ```

- 输入以下内容:wq保存退出，记得填好自己的数据库密码和挂载地址

  ```yaml
  version: "2.1"

  services:
    nextcloud-db:
      image: linuxserver/mariadb
      container_name: nextcloud-db
      restart: unless-stopped
      volumes:
        - /root/nextcloud/db:/config
      environment:
        - PUID=1000
        - PGID=1000
        - TZ=Aisa/Shanghai
        - MYSQL_ROOT_PASSWORD=`数据库ROOT账户密码`
        - MYSQL_PASSWORD=`数据库账户密码`
        - MYSQL_DATABASE=nextcloud
        - MYSQL_USER=nextcloud

    nextcloud:
      image: linuxserver/nextcloud
      restart: unless-stopped
      ports:
        - 9443:443
      container_name: nextcloud
      volumes:
        - /root/nextcloud/config:/config
        - /root/nextcloud/data:/data
        - /root/nextcloud/mnt:/mnt
      environment:
        - UID=1000
        - GID=1000
        - TZ=Aisa/Shanghai
      privileged: true

    phpmyadmin:
      image: linuxserver/phpmyadmin
      container_name: nextcloud-phpmyadmin
      environment:
        - PUID=1000
        - PGID=1000
        - TZ=Aisa/Shanghai
        - PMA_ARBITRARY=1
      #volumes:
      #  - /root/nextcloud/phpmyadmin:/config
      ports:
        - 9980:80
      restart: unless-stopped
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

> 启动完成后，输入你的ip:9443进入nextcloud页面配置。
       部分配置较低的机器（比如我的1C2G）可能会卡住，怎么刷新都卡在了创建用户界面，这时只需要把容器dowm再up就可以了，配置会留在映射的宿主机地址，如果需要完全重新创建，记得去宿主机删除对应的映射文件夹或者换一个地址。

#### **docker-ce + network**

> 容器间采用network链接网络，当然也可以使用--link参数。使用docker-compose是自动创建在同一个网络下。

- 拉取镜像

  ```bash
  #nextcloud
  docker pull linuxserver/nextcloud
  #mariadb
  docker pull linuxserver/mariadb
  #mysql可视化管理工具
  docker pull linuxserver/phpmyadmin
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
  -v /root/nextcloud:/var/www/html \
  -v /root/nextcloud/config:/config
  -v /root/nextcloud/data:/data
  -p 9443:443 \
  -e UID=1000
  -e GID=1000
  -e TZ=Aisa/Shanghai
  linuxserver/nextcloud
  ```

- 启动mariadb容器

  ```bash
  docker run -d \
  --network linuxserver/mariadb \
  --network-alias mariadb \
  --restart=always \
  --name nextcloud_db \
  -v /root/nextcloud/db:/config
  -e PUID=1000
  -e PGID=1000
  -e TZ=Aisa/Shanghai
  -e MYSQL_ROOT_PASSWORD=Dd1
  -e MYSQL_PASSWORD=13245678
  -e MYSQL_DATABASE=nextclou
  -e MYSQL_USER=nextcloud
  linuxserver/mariadb
  ```

- 启动myadmin容器

  ```bash
  docker run -d \
  --network network-nextcloud \
  --network-alias myadmin \
  --restart=always \
  --name nextcloud_db_myadmin \
  -e PUID=1000
  -e PGID=1000
  -e TZ=Aisa/Shanghai
  -e PMA_ARBITRARY=1
  -p 9888:80 \
  linuxserver/phpmyadmin
  ```

- ip:9443进入nextcloud主页，本地ip为localhost,这个镜像默认是不开80端口的，有需要可以进文件夹找nginx文件做代理配置。
配置管理员账户密码，数据库选择mariadb，数据库主机输入mariadb:3306

  ![606AE42C-C8AD-46C4-AADC-3AFCAC26D01D.png](https://img.dycloud.work/images/2021/09/13/606AE42C-C8AD-46C4-AADC-3AFCAC26D01D.png)

- 完成

  ![EA1BC3A3-D6A9-400C-BF4C-B22C33387A75.png](https://img.dycloud.work/images/2021/09/13/EA1BC3A3-D6A9-400C-BF4C-B22C33387A75.png)
 
<!-- tabs:end -->
