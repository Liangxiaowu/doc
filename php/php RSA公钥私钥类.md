#### 简介

现在计算机上使用的非对称加密算法有很多，例如RSA、Elgamal、Rabin等等，但RSA最常用，也是最早的非对称加密算法。

RSA算法是1977年由三位数学家Rivest、Shamir 和 Adleman 设计的一种算法，可以实现非对称加密。RSA算法也是现在最广为使用的"非对称加密算法"。

对RSA原理兴趣的同学推荐去看一下阮一峰的博客：

[RSA的算法原理（一）](http://www.ruanyifeng.com/blog/2013/06/rsa_algorithm_part_one.html)

[RSA的算法原理（二）](http://www.ruanyifeng.com/blog/2013/07/rsa_algorithm_part_two.html)

#### PHP的RSA算法类

##### 演示环境和条件

- laravel5.8
- php7.3.9
- 需要开启php的openssl扩展

##### RSA类封装

```php
class RsaKey
{

    public $publicKey = null;
    public $privateKey = null;

    public function __construct()
    {
      // 获取密钥
        $this->priKey = trim(Storage::get("key/private_key.pem"));

      // 获取公钥
        $this->pubKey = trim(Storage::get("key/public_key.pem"));
      
        // 判断是否开启openssl扩展
        if (!extension_loaded("openssl")) {
          // 异常处理
            throw new RsaKeyException("RSA Error:Please open the openssl extension first",500);
        }
    }


    /**
     * 生成Rsa公钥和私钥
     * @param int $private_key_bits 建议：[512, 1024, 2048, 4096]
     * @return array
     */
    public function createdKeys(int $private_key_bits = 1024)
    {
        $rsa = [
            "private_key" => "",
            "public_key" => ""
        ];

        $config = [
            "digest_alg" => "sha512",
            "private_key_bits" => $private_key_bits,
            "private_key_type" => OPENSSL_KEYTYPE_RSA,
        ];

        //创建公钥和私钥
        $res = openssl_pkey_new($config);

        //提取私钥
        openssl_pkey_export($res, $rsa['private_key']);

        //生成公钥
        $rsa['public_key'] = openssl_pkey_get_details($res)["key"];

        return $rsa;
    }


    /**
     * 生成签名
     * @param string $data 签名材料
     * @return bool|string 签名值
     */
    public function createdSign($data)
    {
        $ret = false;
        if (openssl_sign($data, $result, $this->privateKey)) {
            $ret = base64_encode($result);
        }
        return $ret;
    }



    /**
     * 验证签名
     *
     * @param string $data 签名材料
     * @param string $sign 签名值
     * @return bool
     */
    public function verifySign($data, $sign)
    {
        $ret = false;
      	$sign = base64_decode($sign);
        if ($sign !== false) {
            $ret = (bool)openssl_verify($data, $sign, $this->publicKey);
        }
        return $ret;
    }

    /**
     * 加密
     * @param string $data     明文
     * @param int $padding      填充方式（OPENSSL_PKCS1_PADDING / OPENSSL_NO_PADDING）
     * @return bool|string      密文
     * @throws RsaKeyException
     */
    public function encrypt($data, $padding = OPENSSL_PKCS1_PADDING)
    {
        $ret = false;
        if (openssl_public_encrypt($data, $result, $this->publicKey, $padding)) {
            $ret = base64_decode($result);
        }
        return $ret;
    }


    /**
     * 解密
     * @param string $data       密文
     * @param int $padding      填充方式（OPENSSL_PKCS1_PADDING / OPENSSL_NO_PADDING）
     * @return bool|string      明文
     * @throws RsaKeyException
     */
    public function decrypt($data, $padding = OPENSSL_PKCS1_PADDING)
    {
        $ret = false;
        $data = base64_decode($data);
        if ($data !== false) {
            if (openssl_private_decrypt($data, $result, $this->privateKey, $padding)) {
                $ret = $result.'';
            }
        }
        return $ret;
    }
```

```php
// public_key.pem 公钥文件，可以通过上面RsaKey类中createdKey生成
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDug1l3eC/BWQ8NBqOEirg3pPIh
zkdT66rRMGhEJyw1h4jx9S5zWdszAD+qgZRhW3NZmlC8gvcPw4IHJZi+jgjywnT5
Vhll0MO14mag+Es5+Ot0rEP7G2oEuhyKx89gPIru7cL8ApmOKXUh9x6fuQxObpxX
duoKNkUhLEeJl5LKwDQAB
-----END PUBLIC KEY-----
```

```php
// private_key.pem私钥文件，可以通过上面RsaKey类中createdKey生成
-----BEGIN PRIVATE KEY-----
MIICeAIBADANBgkqhkiG9w0BAQEFAASCAmIwggJeAgEAAoGBAO6DWXd4L8FZDw0G
o4SKuDek8iHOR1PrqtEwaEQnLDWHiPH1LnNZ2zMAP6qBlGFbc1maULyC9w/Dggcl
mL6OCPLCdPlWGWXQw7XiZqD4Szn463SsQ/sbagS6HIrHz2A8iu7twvwCmY4pdSH3
Hp+5DE5unFd26go2RSEsR4mXksqXAgMBAAECgYEAqm9jzBIvFdu8/JLk3/58ew68
E1oi8B30Vz1fFGxlk+7A9h08zyLDlyMzW3TzAcrml33E+aAgSbxsOw0ro+c9DlKQ
h+I5MS4bm2k1U51B0shAg9s1xtsEsK7SaDMPHKlXuUafpFS4sMEkjZxrlhWyF/A3
yDerXjXhIS0b8lbAQNECQQD9e1J7JAIz3O8oAahqo92XJSe1sL3CWMkFW9WIhbdY
11bUMUboPk77Z3i9pcbSRytpLu1e9Ev4hSLiSI9ggMJ/AkEA8OH1KevU3vWanV/Y
JoEwNmWG+CbI3Ti5f19LpLz3I7CbJ1slmzTTbQUXSVRGG+MAiXt4x45URyUclp5n
hjG76QJBAL1/TkAkwUjIaDn+U0u1yVjPPAqGYLZD7bQF3XK8lD7kbVMgq6O/wQgN
aKROpH5bGkb+ABK2i3z0+wkzuYwpdj0CQESbduJrwwOF3HW6WKwO25aCsEB5e3hW
sa6vi7HuCE3n/Sjmcv2TvCwu6QT/tcA3lW2S90GPLj3dOhlEIKQ+AhECQQCKrUzK
ooERDi96AGDBy3guKvvtRxl0vYVwA0B5UAeGcAmiMbwolgCkmY9j3Amo/rljYQS2
3XOwxScTNnpnig
-----END PRIVATE KEY-----
```



