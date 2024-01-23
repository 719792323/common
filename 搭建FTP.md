1. 安装vsftpd

   ```shell
   apt-get install vsftpd -y
   ```

2. 创建用户

   ```she
   # 创建admin用户
   useradd -m admin
   passwd admin
   # 创建非admin用户
   useradd -m songji
   passwd songji
   ```

3. 修改vsftpd.conf配置文件

   ```shell
   vim /etc/vsftpd.conf
   # 添加如下内容
   chroot_local_user=YES
   user_config_dir=/etc/vsftpd_user_list/
   allow_writeable_chroot=YES
   # 设置创建文件的默认权限
   local_umask=000
   ```

4. 配置用户文件

   ```shell
   # 配置admin
   vim /etc/vsftpd_user_list/admin
   local_root=/home/ftp
   write_enable=YES
   # 配置songji
   vim /etc/vsftpd_user_list/songji
   local_root=/home/ftp/songji
   write_enable=YES
   
   ```

5. 创建ftp对应的文件目录

   ```shell
   mkdir /home/ftp
   mkdir /home/ftp/songji
   ```

6. 重启vsftp服务

   ```shell
   systemctl restart vsftpd
   ```

   