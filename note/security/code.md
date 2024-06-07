# ECC

## 密钥转换

### Java

#### `String` -> `PublicKey`

1. X.509 公钥证书通用格式

   ```java
   public static PublicKey recoverPublicKeyFromHexString(String hexPublicKey) throws GeneralSecurityException {
       return KeyFactory.getInstance("EC", "BC")
               .generatePublic(new X509EncodedKeySpec(Hex.decode(hexPublicKey)));
   }
   ```

2. 专用格式（解析时需要指明使用的曲线）

   ```java
   public static PublicKey convertHexToPublicKey(String hexPublicKey) throws GeneralSecurityException {
       final ECParameterSpec sm2ECParameters = ECNamedCurveTable.getParameterSpec("sm2p256v1");
       // 有些密钥解析前要在前面加一个前缀，暂时没有深入研究原因
       final ECPoint point = sm2ECParameters.getCurve().decodePoint(Hex.decode("04" + hexPublicKey));
       return KeyFactory.getInstance("EC", "BC")
               .generatePublic(new ECPublicKeySpec(point, sm2ECParameters));
   }
   ```

#### `String` -> `PrivateKey`

1. PKCS#8 私钥通用格式

   ```java
   public static PrivateKey recoverPrivateKeyFromHexString(String hexString) throws Exception {
       return KeyFactory.getInstance("EC", "BC")
               .generatePrivate(new PKCS8EncodedKeySpec(Hex.decode(hexString)));
   }
   ```

2. 专用格式（解析时需要指明使用的曲线）

   ```java
   public static PrivateKey convertHexToPrivateKey(String hexPrivateKey) throws GeneralSecurityException {
       final ECParameterSpec sm2ECParameters = ECNamedCurveTable.getParameterSpec("sm2p256v1");
       return KeyFactory.getInstance("EC", "BC")
               .generatePrivate(new ECPrivateKeySpec(new BigInteger(hexPrivateKey, 16), sm2ECParameters));
   }
   ```

#### `PublicKey` -> `String`

1. X.509 公钥证书通用格式

   ```java
   final PublicKey publicKey = ...;
   String publicKeyStr = new String(Hex.encode(publicKey.getEncoded()));
   ```

#### `PrivateKey` -> `String`

1. PKCS#8 公钥证书通用格式

   ```java
   final PrivateKey privateKey = ...;
   String publicKeyStr = new String(Hex.encode(privateKey.getEncoded()));
   ```

