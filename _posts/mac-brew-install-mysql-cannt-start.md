title: mac下使用brew安装mysql后无法启动
date: 2013-03-02 16:59:58
tags:
- mac
- mysql
- brew
categroies:
- linux
- tech
---
brew是mac下软件包管理的神器，一直用brew很方便的安装各种软件。最近偷懒想用brew安装下mysql。不料安装完成后遇到了启动不了的情况。错误日志记录如下：  
{% blockquote %}
130227  9:46:12 [Warning] Setting lower_case_table_names=2 because file system for /usr/local/var/mysql/ is case insensitive  
/usr/local/Cellar/mysql/5.5.28/bin/mysqld: Can't find file: './mysql/plugin.frm' (errno: 13)  
130227  9:46:12 [ERROR] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.  
130227  9:46:12 InnoDB: The InnoDB memory heap is disabled  
130227  9:46:12 InnoDB: Mutexes and rw_locks use GCC atomic builtins  
130227  9:46:12 InnoDB: Compressed tables use zlib 1.2.5  
130227  9:46:12 InnoDB: Initializing buffer pool, size = 128.0M  
130227  9:46:12 InnoDB: Completed initialization of buffer pool  
130227  9:46:12  InnoDB: Operating system error number 13 in a file operation.  
**InnoDB: The error means mysqld does not have the access rights to**  
InnoDB: the directory.  
InnoDB: File name ./ibdata1  
InnoDB: File operation call: 'create'.  
InnoDB: Cannot continue operation.  
130227 09:46:12 mysqld_safe mysqld from pid file /usr/local/var/mysql/soonemac.local.pid ended  
{% endblockquote %}
标粗那行表明有权限问题，进去看了下发现错误日志的用户是属于mysql的，其他的所有mysql相关文件的都是属于soone用户的。于是我干脆就把mysql相关的文件的权限改成了777，再次启动mysql，这回就正常了。  
ps:理论上把跟mysql相关的文件和目录所属用户改成mysql应该也是能解决这个问题。
