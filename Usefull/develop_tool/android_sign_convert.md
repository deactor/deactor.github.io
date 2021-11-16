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
