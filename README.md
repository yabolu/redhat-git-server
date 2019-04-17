### Redhat6 搭建Git 服务器

#### 1. 安装Git

###### 1. 更新 yum 源
* 检查yum包 `rpm -qa |grep yum`
* 删除自带包
    `rpm -aq | grep yum | xargs rpm -e –nodeps `
* 再次检查
* 下载更新包(http://mirrors.163.com/centos/6/os/x86_64/Packages/)

    ```
    wget http://mirrors.163.com/centos/6/os/x86_64/Packages/python-urlgrabber-3.9.1-11.el6.noarch.rpm 
    wget http://mirrors.163.com/centos/6/os/x86_64/Packages/yum-plugin-fastestmirror-1.1.30-41.el6.noarch.rpm
    wget http://mirrors.163.com/centos/6/os/x86_64/Packages/yum-3.2.29-81.el6.centos.noarch.rpm 
    wget http://mirrors.163.com/centos/6/os/x86_64/Packages/yum-updateonboot-1.1.30-41.el6.noarch.rpm
    wget http://mirrors.163.com/centos/6/os/x86_64/Packages/yum-metadata-parser-1.1.2-16.el6.x86_64.rpm
    wget http://mirrors.163.com/centos/6/os/x86_64/Packages/yum-utils-1.1.30-41.el6.noarch.rpm
    ```
* 安装
`rpm -ivh yum-* `
如果出现:

    ```
    error: Failed dependencies:
            python-urlgrabber >= 3.9.1-10 is needed by yum-3.2.29-73.el6.centos.noarch
    
    ```
    执行`rpm -Uvh python-urlgrabber-3.9.1-11.el6.noarch.rpm`

* 切换源

    ```
    cd /etc/yum.repos.d/ #进入到yum配置文件目录
    
    wget http://mirrors.163.com/.help/CentOS6-Base-163.repo #下载CentOS配置文件
    ```
    
    编辑配置文件:
    `vi CentOS6-Base-163.repo`
    如果你系统没有配置环境变量 `$releasever` 和 `$basearch`,那么配置文件就把`$releasever` 都改成 6 
    ![](https://github.com/yabolu/redhat-git-server/blob/master/163-repo.jpg)
    
    ```
    yum clean all
    yum makecache
    ```
    
###### 2. 安装git

```
wget https://www.kernel.org/pub/software/scm/git/git-1.9.4.tar.gz`
tar zxf git-1.9.4.tar.gz
cd git-1.9.4
./configure
make
make install
```

如果安装过程出现错误：

* 错误1
    
    ```
    In file included from credential-store.c:1:
    cache.h:21:18: warning: zlib.h: No such file or directory
    In file included from credential-store.c:1:
    cache.h:23: error: expected specifier-qualifier-list before ‘z_stream’
    make: *** [credential-store.o] Error 1
    ```
        
    解决：
        
    ```
    yum install zlib
    yum install zlib-devel
    ```
    
* 错误2

    ```
    usr/bin/perl Makefile.PL PREFIX='/usr/local/git' '' --localedir='/usr/local/git/share/locale'Can't locate ExtUtils/MakeMaker.pm in @INC (@INC contains: /usr/local/lib64/perl5 /usr/local/share/perl5 /usr/lib64/perl5/vendor_perl /usr/share/perl5/vendor_perl /usr/lib64/perl5 /usr/share/perl5 .) at Makefile.PL line 3.BEGIN failed--compilation aborted at Makefile.PL line 3.make[1]: *** [perl.mak] Error 2 make: *** [perl/perl.mak] Error 2 
    ```
    
    解决:
    
    ```
    yum install perl-ExtUtils-MakeMaker package
    service httpd restart
    ```

#### 2. 搭建Git服务器

* 确保 `git` 安装成功 `git --version`
* 创建账号

    ```
    sudo adduser git
    passwd git your-passwd
    ```
* 打开RSA认证
    `Git` 服务器打开 `RSA` 认证 
    
    ```
    cd /etc/ssh 
    cat sshd_config
    确保:
    1.RSAAuthentication yes 
    2.PubkeyAuthentication yes
    3. AuthorizedKeysFile  .ssh/authorized_keys  //说明公钥放在.ssh/authorized_keys 文件中
    
    cd /home/git
    mkdir .ssh
    touch authorized_keys  
    ```
    
    收集所有需要登录的用户的公钥，就是他们自己的 `id_rsa.pub` 文件，把所有公钥导入到 `/home/git/.ssh/authorized_keys` 文件里，一行一个
    
* 初始化 `Git` 仓库

    1. 先选定一个目录作为 `Git` 仓库，假定是 `/srv/sample.git`，在 `/srv` 目录下输入命令
    `git init --bare sample.git`
    
    2. 把 `owner` 改为 `git`, `sudo chown -R git:git sample.git`
    3. 通过 `git clone` 命令克隆远程仓库了，在各自的电脑上运行：
        `git clone git@server:/srv/sample.git`
        
        
        


#### 参考文章
1. 更新 `yum` 源
    1. [RedHat6.5 更换Yum源](https://blog.csdn.net/u011641865/article/details/78518214)
    2. [RedHat6.6 enterprise 更新国内CentOS Yum源](http://www.qedev.com/linux/93.html)
2. 安装Git
    1. [Git 在RedHat安装](https://blog.csdn.net/superman_xing/article/details/78737390)

3. 搭建Git服务器
    1. [搭建Git服务器](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00137583770360579bc4b458f044ce7afed3df579123eca000)
    2. [Linux搭建git服务器，自建仓库、本地和Windows下面clone](https://www.jianshu.com/p/8fa1c989259b)


