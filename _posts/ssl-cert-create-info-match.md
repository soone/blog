title: ssl证书生成信息匹配问题
date: 2013-03-21 14:52:55
tags:
- ssl
categories:
- tech
---
今天财务说她内网老服务器的ssl证书过期了无法登陆window服务器了。过期后重新生成证书是必须的。不过发现我没老服务器的根证书，幸亏今天zj来了(我们的老员工)，否则就杯具了。。  
拿到证书后开始在fedora上做几个简单步骤的证书生成：
{% codeblock %}
openssl genrsa -des3 -out liny.key 2048  
openssl rsa -in liny.key -out liny.key  
openssl req -new -key liny.key -out liny.csr  
# 注意下面这个语句，如果你的openssl.cnf是用的默认的，那可能是在  
# /etc/pki/tls/下面，里面默认配置的相应目录是在/etc/pki/CA目录下，  
# 这里面会要求在CA下有个index.txt和serial的文件，如果不存在请手动  
# 创建，同时在index.txt里面写入7C，这是拍脑袋的文字，随便写  
sudo openssl ca -in liny.csr -out liny.crt -cert kfs.crt -keyfile kfs.key  
# 上面的这个命令执行的时候总是出现这样的提示：
# Using configuration from /etc/pki/tls/openssl.cnf  
# Check that the request matches the signature  
# Signature ok  
# The stateOrProvinceName field needed to be the same in the  
# CA certificate (Fujian) and the request (Fujian)  
# 明显我填的身份是对的，可是这里总是提示是错误的。
{% endcodeblock %}
这个纠结的问题后来在stackoverflow上找到了答案，需要修改openssl.cnf里面的string_mask的值，将**utf8only**修改成**pkix**即可。  

参考地址：http://stackoverflow.com/questions/6976127/openssl-signing-a-certificate-with-my-ca  
