---
title: '[CentOS 7] SVN Server 架設'
date: 2018-01-07 16:38:09
categories:
- [Linux]
tags:
- Linux
- Subversion
---

**前言**   
   
這個架設筆記使用的環境是 CentOS 7， 在 CentOS 7 中的 subversion 版本比較低，所以另外尋找其他的 repository 來安裝，在這選擇的版本是 1.8，登入驗證的部分是設定跟 AD 認證 (也可以跟 ldap，之後再補充)

<!--more-->
   
**安裝**   

-  準備 subversion 來源   
    -  在 /etc/yum.repos.d/ 下建立一個 repo 檔   
        ```sh
        cd /etc/yum.repos.d
        vi CentOS-SVN.repo
        ```

    - 貼上以下內容後存檔   
        ```sh
        [WandiscoSVN]
        name=Wandisco SVN Repo
        baseurl=http://opensource.wandisco.com/centos/7/svn-1.8/RPMS/$basearch/
        enabled=1
        gpgcheck=1
        ```

    - 完成後執行一下 update   
        ```sh
        yum update
        ```
- 安裝需要的套件
    - apache   
        ```sh
        yum install httpd
        ```

    - subversion 及需要的套件   
        ```sh
        yum install subversion mod_dav_svn
        ```

    -  apache ldap module (SVN 的認證是跟 AD 或 LDAP 認證)   
        ```sh
        yum install mod_ldap
        ```
   
**配置**   

- 建立放檔案庫的目錄   
    ```sh
    cd /opt
    mkdir -p svn/svnroot
    chown -R apache:apache svn
    ```

- 編輯設定檔   
    ```sh
    cd /etc/httpd/conf.d
    vi subversion.conf
    ```

- 修改 檔案裡的 <Location /svn> 區塊   
    ```sh
    <Location /svn>
        DAV svn
        SVNParentPath /opt/svn/svnroot
        AuthBasicProvider ldap
        AuthType Basic
        AuthName "SVN Auth. (AD)"
        AuthLDAPBindDN "登入AD的DN"
        AuthLDAPBindPassword "登入AD的密碼"
        AuthDLAPURL "  ldap://xxx.xxx.xxx.xxx:3268/根節點?sAMAccountName?sub?(objectClass=user)"
        AuthzForceUsernameCase lower
        AuthzSVNReposRelativeAccessFile authz
        Require valid-user
        Allow from all
    </Location>
    ```

- 關閉 SELinux   
    - 編輯 /etc/sysconfig/selinux   
        ```sh
        vi /etc/sysconfig/selinux
        ```

    - 將 SELINUX  改為 disabled，並註解 SELINUXTYPE  
 
        ```sh
        # This file controls the state of SELinux on the system.
        # SELINUX= can take one of these three values:
        #     enforcing - SELinux security policy is enforced.
        #     permissive - SELinux prints warnings instead of enforcing.
        #     disabled - No SELinux policy is loaded.
        SELINUX=disabled
        # SELINUXTYPE= can take one of three two values:
        #     targeted - Targeted processes are protected,
        #     minimum - Modification of targeted policy. Only selected processes are protected.
        #     mls - Multi Level Security protection.
        #SELINUXTYPE=targeted
        ```

- 防火牆設定   
    ```sh
    firewall-cmd -add-port=80/tcp --permanent
    firewall-cmd --reload
    ```

- 設定隨開機啟動   
    ```sh
    systemctl enable httpd.service
    ```

- 執行服務   

    ```sh
    systemctl start httpd.service
    ```
   
**後記**   

到這邊都還順利的話，算是已經 將服務架設好了，在 /opt/svn/svnroot/ 下透過 svn 指令建立 repository 和設定權限後即可開始使用。   
   