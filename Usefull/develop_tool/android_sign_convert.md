---
title:  "Android 签名文件转换"
---
### 系统签名转keystore
本文在ubuntu下使用openssl转换，三步：
1. openssl pkcs8 -in platform.pk8 -inform DER -outform PEM -out shared.priv.pem -nocrypt
2. openssl pkcs12 -export -in platform.x509.pem -inkey shared.priv.pem -out shared.pk12 -name testalias (其中testalias为keyAlias，可以自己命名，如使用platform)
3. keytool -importkeystore -deststorepass android -destkeypass android -destkeystore source.keystore -srckeystore shared.pk12 -srcstoretype PKCS12 -srcstorepass android -alias keyAlias (命令中的android为密码，即第二步中输入的密码，最后一个keyAlias为上一步中指定的name，如platform，-destkeystore指定输出的keyStore文件名)

如将platform.pk8转为platform.keystore, keyAlias和password全部设置为platform。命令如下：
1. openssl pkcs8 -in platform.pk8 -inform DER -outform PEM -out shared.priv.pem -nocrypt
2. openssl pkcs12 -export -in platform.x509.pem -inkey shared.priv.pem -out shared.pk12 -name platform 
3. keytool -importkeystore -deststorepass platform -destkeypass platform -destkeystore platform.keystore -srckeystore shared.pk12 -srcstoretype PKCS12 -srcstorepass platform -alias platform 

### 手动签名
jarsigner -keystore E:\platform.jks -signedjar app_signed.apk app_unsigned.apk platform
+ jarsigner：jdk中带的工具。
+ E:\platform.jks：签名文件路径
+ platform：签名文件别名（alias）

输入命令后会提示输入密码。填写正确签名密码即可进行签名。  

如果提示"java.util.zip.ZipException: invalid entry compressed size"则说明app_unsigned.apk是已经签过名的。需要把已经签的签名给去掉。
app_unsigned.apk改app_unsigned.zip，将META-INF目录删除，重新将zip换成apk即可。
