# rsa-key-switch
RSA证书PKCS1与PKCS8之间的互验互签解决方案 Java的pkcs8证书与python等的pcks1证书互验互签

### 前眼
  最近做了一个用到加密、加签验签的项目,对这块有了一些新的认识,分享记录一下.</br>
  先说一下我理解下的这几个词的概念:</br>
  ```RSA```是一种非对称加密算法,```RSA证书```,自然也就是基于这种算法生成的证书.</br>
  ```PEM```格式证书文件,这个我理解的就是base64展示形式的证书内容文件,为什么说一下这个,因为你的合作方,可能给你各种格式/表现形式的证书,你需要转换为你的系统里的通用格式;</br>
  ```pkcs8,pkcs1等```是RSA证书标准,还有其他型号的,这里指列举两种标准,其他标准用同样的方法也是可以的.

### 第一步:转换格式
  不管你拿到的是什么格式什么标准的RSA证书格式,只要内容是匹配的,就可以正常转换.</br>
  如果是拿到的pem等格式,可以直接使用```openssl```进行转换.比如</br>
  PKCS8格式私钥转换为PKCS1格式： 
  ``` shell 
    openssl rsa -in pkcs8.pem -out pkcs1.pem 
  ```  
  </br>
  其他转换命令网上还有很多.比如</br>
   http://blog.csdn.net/weixin_34071713/article/details/93242506
  </br>
  如果你拿到的,不是直接的这些格式,比如就是一个文件流?那也不要紧,只要把文件流读出来,以base64格式的形式打印出来,自己生成pem文件:  </br>
  比如拿到的是一个字节流文件: </br>
  ```Python
    def get_base64_key():
      filename = project_dir + "conf/dev/private_test"
      buf = bytearray(os.path.getsize(filename))
      with open(filename, 'rb') as f:
          (f.readinto(buf))
      print(base64.b64encode(buf))
  ```
  
  这样得到这个串之后,再自己创建pem文件,把内容放进去,要注意的是,需要把头和尾加上,比如pkcs8需要加:</br>
  头:```diff +
    -----BEGIN PRIVATE KEY----- 
    ```  </br>
  尾:```diff +
    -----END PRIVATE KEY----- 
    ```  </br>
