# rclone挂载OneDrive

> 阿里云轻量服务器的存储只有40G，本想着搭建完nextcloud后挂载各大oss，参考了一番各大收费，感觉还是白嫖才是最香的，就想到了我的E5账号，一个账号5T，25个那就是125T，都能把我装进去了。冲浪了一番，发现全是rclone的方案，啪一下，就复制粘贴过来了。

September 11, 2021 [NextCloud](/Docker/Nextcloud.md) 

---

## 部署Rclone

- 用的是CentOS7，配置还需要一台windows电脑。首先是下载，centos就一条命令。

    ```bash
    curl https://rclone.org/install.sh | sudo bash
    ```

- 开始配置，配置文件生成在/root/.config，这里以windows为例，linux命令也一样。

    ```bash
    rclone config
    ```

- 接下来按着步骤走，选择自己需要挂载的网盘，还挺多的，不一定是OneDrive

    ```bash
    No remotes found - make a new one
    n) New remote                 
    s) Set configuration password 
    q) Quit config
    n/s/q> n         #这里输入n新建
    name> OneDrive   #命名，自己喜欢的
    Type of storage to configure.
    Enter a string value. Press Enter for the default ("").
    Choose a number from below, or type in your own value
     1 / 1Fichier
       \ "fichier"
    ...略
    26 / Microsoft OneDrive
       \ "onedrive"
    ...略
    43 / seafile
       \ "seafile"
    Storage> 26  #选择需要挂载网盘的序号，我这里的OneDrive是26
    OAuth Client Id
    Leave blank normally.
    Enter a string value. Press Enter for the default ("").
    client_id> #跳过，如果有也可以输入，不知道是什么就跳过
    OAuth Client Secret
    Leave blank normally.
    Enter a string value. Press Enter for the default ("").
    client_secret> #跳过，如果有也可以输入，不知道是什么就跳过
    Choose national cloud region for OneDrive.
    Enter a string value. Press Enter for the default ("global").
    Choose a number from below, or type in your own value
     1 / Microsoft Cloud Global
       \ "global"
     2 / Microsoft Cloud for US Government
       \ "us"
     3 / Microsoft Cloud Germany
       \ "de"
     4 / Azure and Office 365 operated by 21Vianet in China
       \ "cn"
    region> 1  #我是E5属于国际版所以是1，国内家庭版的选4
    Edit advanced config?
    y) Yes
    n) No (default)
    y/n>  #回车
    Use auto config?
     * Say Y if not sure
     * Say N if you are working on a remote or headless machine

    y) Yes (default)
    n) No
    y/n>  #windows下回车。centos下n，需要手动输入token
    #这里会跳转网站登录，授权就可以了
    2021/09/11 17:55:06 NOTICE: If your browser doesn't open automatically go to the following link: http://127.0.0.1:53682/auth?state=CwmsKEOEvB4K__ye21ORcw
    2021/09/11 17:55:06 NOTICE: Log in and authorize rclone for access
    2021/09/11 17:55:06 NOTICE: Waiting for code...
    2021/09/11 17:55:12 NOTICE: Got code
    Type of connection
    Enter a string value. Press Enter for the default ("onedrive").
    Choose a number from below, or type in an existing value
     1 / OneDrive Personal or Business
       \ "onedrive"
    ...略
     7 / Sharepoint server-relative path (advanced, e.g. /teams/hr)
       \ "path"
    config_type> 1 #不懂是什么就选1
    Drive OK?

    Found drive "root" of type "business"
    URL: #不给看，是你的OneDrive地址

    y) Yes (default)
    n) No
    y/n>
    --------------------
    [OneDrive]
    type = onedrive
    #这里会一大段token，需要centos使用的要复制包括{}的所有内容
    token = {"access_token":"eyv略08:00"}
    drive_id = b!WHY9d6V2Ukqt37NwJ8t8rCG9CwkEtZlIirpe9uB__TZQzKbEJjcrR6wOYrTQexQn
    drive_type = business
    --------------------
    y) Yes this is OK (default)
    e) Edit this remote
    d) Delete this remote
    y/e/d> #回车
    Current remotes:
    #出现这个已经OK了
    Name                 Type
    ====                 ====
    OneDrive             onedrive

    e) Edit existing remote
    n) New remote
    d) Delete remote
    r) Rename remote
    c) Copy remote
    s) Set configuration password
    q) Quit config
    e/n/d/r/c/s/q> q #退出
    ```

- 挂载，以windows为例，Linux相同。win下挂载目录不能存在，而linux挂载需要提前建好目录。

    ```bash
    #将OneDrive根目录挂载到E盘符上
    rclone mount onedrive:/ E: --vfs-cache-mode writes
    ```

## 设置开机自动挂载

---

### 开机执行单命令

    ```bash
    chmod +x /etc/rc.d/rc.local #更改权限
    vim /etc/rc.d/rc.local
    ```

- 写入开机命令，:wq退出

    ```bash
    rclone mount OneDrive:/ /root/Onedrive_dycloud \
    --copy-links --no-gzip-encoding \
    --no-check-certificate \
    --allow-other \
    --allow-non-empty \
    --vfs-cache-mode writes \
    --vfs-cache-max-age 3h |
    ```

### 设置开机服务，适合多条挂载

- 进入服务目录

    ```bash
    cd /etc/systemd/system
    ```

- 新建rclone@service

    ```bash
    vim rclone@service
    ```

- 写入指令

    ```bash
    [Unit]
    Description=rclone mount %I drive
    After=network.target

    [Service]
    #Type=notify
    Type=simple
    #PrivateTmp=true
    ExecStart=/usr/bin/rclone mount %i: /root/%i --copy-links --no-gzip-encoding --no-check-certificate --allow-other --allow-non-empty --vfs-cache-mode writes --config /root/.config/rclone/rclone.conf

    [Install]
    WantedBy=multi-user.target
    ```

- :wq退出，直接命令行执行守护shell

    ```bash
    for a in `grep '^\[' /root/.config/rclone/rclone.conf`
    do
        b=${a:1:-1}
        [ ! -d "/root/${b}" ] && mkdir /root/${b}
        systemctl enable rclone@${b}
        systemctl start rclone@${b}
    done
    ```

- 以后添加挂载的命令

    ```bash
    mkdir /root/`目录` #创建
    systemctl enable rclone@`remote` #开机执行
    systemctl start rclone@`remote` #马上执行
    ```

### 卸载命令
    ```bash
    fusermount -qzu `目标文件夹`
    ```