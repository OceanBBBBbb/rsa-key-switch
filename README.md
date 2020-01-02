# rsa-key-switch
RSA证书PKCS1与PKCS8之间的互验互签解决方案 Java的pkcs8证书与python等的pcks1证书互验互签

### 前眼
  最近做了一个用到加密、加签验签的项目,对这块有了一些新的认识,分享记录一下.</br>
  先说一下我理解下的这几个词的概念:</br>
  ```RSA```是一种非对称加密算法,```RSA证书```,自然也就是基于这种算法生成的证书.</br>
  ```PEM```格式证书文件,这个我理解的就是base64展示形式的证书内容文件,为什么说一下这个,因为你的合作方,可能给你各种格式/表现形式的证书,你需要转换为你的系统里的通用格式;</br>
  ```pkcs8,pkcs1等```是RSA证书标准,还有其他型号的,这里指列举两种标准,其他标准用同样的方法也是可以的.

### 转换格式
  不管你拿到的是什么格式什么标准的RSA证书格式,只要内容是匹配的,就可以正常转换.</br>
  如果是拿到的pem等格式,可以直接使用```openssl```进行转换.比如</br>
  PKCS8格式私钥转换为PKCS1格式： 
  ``` shell 
    openssl rsa -in pkcs8.pem -out pkcs1.pem 
  ```  
  其他转换命令网上还有很多.比如(:interrobang:这么多出不来的?)
  [链接](http://blog.csdn.net/weixin_34071713/article/details/93242506)
  </br>
  如果你拿到的,不是直接的这些格式,比如就是一个文件流?那也不要紧,只要把文件流读出来,以base64格式的形式打印出来,自己生成pem文件:  </br>
  比如拿到的是一个字节流文件: </br>
  ``` Python
    def get_base64_key():
      filename = project_dir + "conf/dev/private_test"
      buf = bytearray(os.path.getsize(filename))
      with open(filename, 'rb') as f:
          (f.readinto(buf))
      print(base64.b64encode(buf))
  ```
  
  这样得到这个串之后,再自己创建pem文件,把内容放进去,要注意的是,需要把头和尾加上,比如pkcs8需要加:</br>
  头:``` -----BEGIN PRIVATE KEY----- ```  </br>
  尾:``` -----END PRIVATE KEY----- ```</br>
  不然做转换的时候openssl会报错,其他标准的可以查下他的开头结尾解析标识,自行加上就好;
  
 
### 使用
  代码中的例子,即用私钥文件进行加签,公钥文件进行验签,一般加签时,会先将要加签的参数进行加密,然后再加签,参数内容过长,会影响加签,所以基本都会这样操作,不排除有一些可能直接拿原始参数加签的,如果你是做验签方,那么你需要知道对方加签时是怎么做的,采用什么进行的加密,又是按什么样的参数规则、顺序等,这些都要一致,不然也是会验签不过的.</br>
  如python中,按参数的key的assic码排序后,进行```SHA256```加密后加签:
  ``` Python
  with open(private_key_pkcs1, 'r') as f:
    private_key = rsa.PrivateKey.load_pkcs1(f.read().encode())
    
  # 私钥加签
  def sign_msg(msg):
    # 我们是先对字符进行sha256加密，再给密文进行了加签
    ec = sha256_encrypt(msg)
    orig = base64.b64encode(rsa.sign(bytes.fromhex(ec), private_key, 'SHA-256'))
    return orig.decode('utf-8')
    
  def sha256_encrypt(msg):
    sha256 = hashlib.sha256()
    sha256.update(msg.encode('utf-8'))
    res = sha256.hexdigest()
    # print("sha256加密结果:", res)
    return res

  ```
  在Java中进行验签:
  ``` Java
  public static boolean verifySign(PublicKey publicKey, String plain_text, String signed) {
        MessageDigest messageDigest;
        boolean signedSuccess = false;
        try {
            messageDigest = MessageDigest.getInstance("SHA-256");
            messageDigest.update(plain_text.getBytes());
            byte[] outputDigest_verify = messageDigest.digest();
            Signature verifySign = Signature.getInstance("SHA256withRSA");
            verifySign.initVerify(publicKey);
            verifySign.update(outputDigest_verify);
            signedSuccess = verifySign.verify(Base64.decodeBase64(signed));
        } catch (NoSuchAlgorithmException | InvalidKeyException | SignatureException e) {
            log.error("", e);
        }
        return signedSuccess;
    }
  ```
  只要是同一对证书转换来的,python就用它自带的pkcs1的私钥进行加签,java也用它自带的pkcs8加签,用它自己的X509标准公钥验签.
  具体可以看一下完整的代码.
