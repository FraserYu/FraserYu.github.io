---
title: PKIX证书导入
categroies: [Coding, 安全-协议]
tags:
  - SSL
id: pkix-certificate-import
date: 2019-04-12 11:55:47
---

<blockquote class="blockquote-center">通过理解SSL握手过程解决客户端证书问题</blockquote>



最近做项目，调用远程服务器突然出现如下错误：

```java
PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
```



很明显，客户端找不到服务器端返回的public certificate，所以我们需要拿到服务器给客户端的证书，并且存放到Keystore中，这就涉及到SSL握手（handshake）获取信任的过程。

### 图解SSL（Secure Socket Layer）协议

开始加密通信之前，客户端和服务器首先必须建立连接和交换参数，这个过程叫做握手（handshake）
<!-- more -->

<img itemprop="url image" src="/uploads/SSL.png"/>



1. 客户端给出协议版本号、一个客户端生成的随机数（Client random），以及客户端支持的加密方法
2. 服务端确认双方使用的加密方法，并给出数字证书、以及一个服务器生成的随机数（Server random）
3. 客户端确认数字证书有效，然后生成一个新的随机数（Premaster secret），并使用数字证书中的公钥，加密这个随机数，发给服务端
4. 服务端使用自己的私钥，获取爱丽丝发来的随机数（即Premaster secret）
5. 客户端和服务器根据约定的加密方法，使用前面的三个随机数，生成"对话密钥"（session key），用来加密接下来的整个对话过程





以上就是握手的整个过程，其中握手阶段有三点需要注意

1. 生成对话密钥一共需要三个随机数。
2. 握手之后的对话使用"对话密钥"加密（对称加密），服务器的公钥和私钥只用于加密和解密"对话密钥"（非对称加密RSA），无其他作用。
3. 服务器公钥放在服务器的数字证书之中。

### SSL具体握手过程

用-Djavax.net.debug=all来开启调试信息的输出，SSL的握手过程一目了然：

```java
2018-02-01 17:05:24 --INFO-- ***Helper : https://xxx.xxx.com:8443/***/ws/***Service
Ignoring unsupported cipher suite: TLS_DHE_DSS_WITH_AES_128_CBC_SHA256
Ignoring unsupported cipher suite: TLS_DHE_DSS_WITH_AES_256_CBC_SHA256
Ignoring unsupported cipher suite: TLS_DHE_RSA_WITH_AES_128_CBC_SHA256
Ignoring unsupported cipher suite: TLS_ECDH_RSA_WITH_AES_128_CBC_SHA256
Ignoring unsupported cipher suite: TLS_DHE_RSA_WITH_AES_256_CBC_SHA256
Ignoring unsupported cipher suite: TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
Ignoring unsupported cipher suite: TLS_ECDH_ECDSA_WITH_AES_256_CBC_SHA384
Ignoring unsupported cipher suite: TLS_RSA_WITH_AES_256_CBC_SHA256
Ignoring unsupported cipher suite: TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
Ignoring unsupported cipher suite: TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
Ignoring unsupported cipher suite: TLS_ECDH_RSA_WITH_AES_256_CBC_SHA384
Ignoring unsupported cipher suite: TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
Ignoring unsupported cipher suite: TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA256
Ignoring unsupported cipher suite: TLS_RSA_WITH_AES_128_CBC_SHA256
Allow unsafe renegotiation: false
Allow legacy hello messages: true
Is initial handshake: true
Is secure renegotiation: false
%% Client cached [Session-5, TLS_RSA_WITH_AES_256_CBC_SHA]
%% Try resuming [Session-5, TLS_RSA_WITH_AES_256_CBC_SHA] from port 12371
##客户端跟服务器第一次打招呼##
*** ClientHello, TLSv1
RandomCookie:  GMT: 1517410388 bytes = { 95, 120, 132, 216, 69, 34, 205, 73, 133, 30, 240, 178, 37, 187, 228, 127, 73, 149, 1, 216, 19, 3, 35, 236, 29, 111, 125, 132 }
Session ID:  {248, 40, 73, 62, 46, 20, 43, 249, 49, 73, 148, 196, 222, 200, 202, 68, 221, 241, 33, 57, 128, 93, 149, 1, 7, 112, 248, 66, 173, 203, 187, 1}
##支持的加密算法##
Cipher Suites: [TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA, TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA, TLS_RSA_WITH_AES_256_CBC_SHA, TLS_ECDH_ECDSA_WITH_AES_256_CBC_SHA, TLS_ECDH_RSA_WITH_AES_256_CBC_SHA, TLS_DHE_RSA_WITH_AES_256_CBC_SHA, TLS_DHE_DSS_WITH_AES_256_CBC_SHA, TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA, TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA, TLS_RSA_WITH_AES_128_CBC_SHA, TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA, TLS_ECDH_RSA_WITH_AES_128_CBC_SHA, TLS_DHE_RSA_WITH_AES_128_CBC_SHA, TLS_DHE_DSS_WITH_AES_128_CBC_SHA, TLS_ECDHE_ECDSA_WITH_3DES_EDE_CBC_SHA, TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA, SSL_RSA_WITH_3DES_EDE_CBC_SHA, TLS_ECDH_ECDSA_WITH_3DES_EDE_CBC_SHA, TLS_ECDH_RSA_WITH_3DES_EDE_CBC_SHA, SSL_DHE_RSA_WITH_3DES_EDE_CBC_SHA, SSL_DHE_DSS_WITH_3DES_EDE_CBC_SHA, TLS_ECDHE_ECDSA_WITH_RC4_128_SHA, TLS_ECDHE_RSA_WITH_RC4_128_SHA, SSL_RSA_WITH_RC4_128_SHA, TLS_ECDH_ECDSA_WITH_RC4_128_SHA, TLS_ECDH_RSA_WITH_RC4_128_SHA, SSL_RSA_WITH_RC4_128_MD5, TLS_EMPTY_RENEGOTIATION_INFO_SCSV]
Compression Methods:  { 0 }
Extension elliptic_curves, curve names: {secp256r1, sect163k1, sect163r2, secp192r1, secp224r1, sect233k1, sect233r1, sect283k1, sect283r1, secp384r1, sect409k1, sect409r1, secp521r1, sect571k1, sect571r1, secp160k1, secp160r1, secp160r2, sect163r1, secp192k1, sect193r1, sect193r2, secp224k1, sect239k1, secp256k1}
Extension ec_point_formats, formats: [uncompressed]
Extension server_name, server_name: [host_name: idmtest.dcpc.com]
***
[write] MD5 and SHA1 hashes:  len = 220
0000: 01 00 00 D8 03 01 5A 72   D8 54 5F 78 84 D8 45 22  ......Zr.T_x..E"
0010: CD 49 85 1E F0 B2 25 BB   E4 7F 49 95 01 D8 13 03  .I....%...I.....
...

DefaultQuartzScheduler_Worker-3, WRITE: TLSv1 Handshake, length = 220
[Raw write]: length = 225
0000: 16 03 01 00 DC 01 00 00   D8 03 01 5A 72 D8 54 5F  ...........Zr.T_
0010: 78 84 D8 45 22 CD 49 85   1E F0 B2 25 BB E4 7F 49  x..E".I....%...I
...

[Raw read]: length = 5
0000: 16 03 01 06 22                                     ...."
[Raw read]: length = 1570
0000: 02 00 00 46 03 01 5A 72   63 4E F2 CF 4B 0F CE 58  ...F..ZrcN..K..X
0010: 0E EF 36 D4 F3 3E C0 85   AD 6D 4B 8D DB CA FB 77  ..6..>...mK....w
0020: 80 B3 A1 D2 A7 90 20 D5   A6 FD 6E A7 FF 5A D4 DF  ...... ...n..Z..
...

DefaultQuartzScheduler_Worker-3, READ: TLSv1 Handshake, length = 1570

##服务器响应
*** ServerHello, TLSv1
RandomCookie:  GMT: 1517445966 bytes = { 242, 207, 75, 15, 206, 88, 14, 239, 54, 212, 243, 62, 192, 133, 173, 109, 75, 141, 219, 202, 251, 119, 128, 179, 161, 210, 167, 144 }
Session ID:  {213, 166, 253, 110, 167, 255, 90, 212, 223, 9, 221, 44, 82, 61, 249, 50, 79, 110, 177, 174, 106, 198, 101, 47, 213, 191, 119, 128, 106, 200, 188, 1}
##服务器选定的加密算法
Cipher Suite: TLS_RSA_WITH_AES_256_CBC_SHA
Compression Method: 0
***
Warning: No renegotiation indication extension in ServerHello
%% Initialized:  [Session-8, TLS_RSA_WITH_AES_256_CBC_SHA]
** TLS_RSA_WITH_AES_256_CBC_SHA
[read] MD5 and SHA1 hashes:  len = 74
0000: 02 00 00 46 03 01 5A 72   63 4E F2 CF 4B 0F CE 58  ...F..ZrcN..K..X
0010: 0E EF 36 D4 F3 3E C0 85   AD 6D 4B 8D DB CA FB 77  ..6..>...mK....w
...

##服务器返回给客户端的证书链,证明“我是我”
*** Certificate chain
chain [0] = [
[
  Version: V3
  Subject: CN=*.****.com, ST=北京市, L=北京市, O=***公司, C=CN
  Signature Algorithm: SHA256withRSA, OID = 1.2.840.113549.1.1.11

  Key:  Sun RSA public key, 2048 bits
  modulus: 256891288892062675553247103925...
  public exponent: 65537
  Validity: [From: Wed May 17 16:15:30 CST 2017,
               To: Fri May 15 16:15:30 CST 2020]
  Issuer: CN=WoSign OV SSL CA, O=WoSign CA Limited, C=CN
  SerialNumber: [    363fa17d 5c46082c dc707c24 6a8d81b2]

Certificate Extensions: 9
[1]: ObjectId: 1.3.6.1.5.5.7.1.1 Criticality=false
AuthorityInfoAccess [[
   accessMethod: ocsp
   accessLocation: URIName: http://wosign-ovca.ocsp-certum.com,
   accessMethod: caIssuers
   accessLocation: URIName: http://repository.certum.pl/wosign-ovca.cer]]

[2]: ObjectId: 2.5.29.35 Criticality=false
AuthorityKeyIdentifier [
KeyIdentifier [
0000: A1 13 54 DC 56 73 2C 27   82 CA C8 84 EF EE BF 00  ..T.Vs,'........
0010: FD 5F AB 56                                        ._.V
]]

[3]: ObjectId: 2.5.29.19 Criticality=true
BasicConstraints:[
  CA:false
  PathLen: undefined
]

[4]: ObjectId: 2.5.29.31 Criticality=false
CRLDistributionPoints [
  [DistributionPoint:
     [URIName: http://wosign.crl.certum.pl/wosign-ovca.crl]
]]

[5]: ObjectId: 2.5.29.32 Criticality=false
CertificatePolicies [
  [CertificatePolicyId: [2.23.140.1.2.2]
[]  ]
  [CertificatePolicyId: [1.2.616.1.113527.2.5.1.12.2]
[PolicyQualifierInfo: [
  qualifierID: 1.3.6.1.5.5.7.2.2
  qualifier:
0000: 30 81 E4 30 1F 16 18 41   73 73 65 63 6F 20 44 61  0..0...Asseco Da
0010: 74 61 20 53 79 73 74 65   6D 73 20 53 2E 41 2E 30  ta Systems S.A.0
...

]]  ]
]

[6]: ObjectId: 2.5.29.37 Criticality=false
ExtendedKeyUsages [
  serverAuth
  clientAuth
]

[7]: ObjectId: 2.5.29.15 Criticality=true
KeyUsage [
  DigitalSignature
  Key_Encipherment
]

[8]: ObjectId: 2.5.29.17 Criticality=false
SubjectAlternativeName [
  DNSName: *.dcpc.com
  DNSName: dcpc.com
]

[9]: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: B3 A6 53 AD 96 D8 B7 51   09 D5 E1 07 E5 B2 BC E6  ..S....Q........
0010: B5 F2 82 2B                                        ...+
]]]
  Algorithm: [SHA256withRSA]
  Signature:
0000: 9A 38 A4 8F B2 A8 CE 9A   B6 98 2F 8F 18 AA 45 79  .8......../...Ey
0010: DE D6 31 22 99 6E D7 14   45 5E C1 1F 06 EC 7F C9  ..1".n..E^......
...

]
***
==客户端从本地找到的信赖证书(如果找不到会直接报certificate_unknow)==
Found trusted certificate:
[
[
  Version: V3
  Subject: CN=*.****.com, ST=北京市, L=北京市, O=****公司, C=CN
  Signature Algorithm: SHA256withRSA, OID = 1.2.840.113549.1.1.11

  Key:  Sun RSA public key, 2048 bits
  modulus: 2568912888920626755532471039254124....
  public exponent: 65537
  Validity: [From: Wed May 17 16:15:30 CST 2017,
               To: Fri May 15 16:15:30 CST 2020]
  Issuer: CN=WoSign OV SSL CA, O=WoSign CA Limited, C=CN
  SerialNumber: [    363fa17d 5c46082c dc707c24 6a8d81b2]

Certificate Extensions: 9
[1]: ObjectId: 1.3.6.1.5.5.7.1.1 Criticality=false
AuthorityInfoAccess [
  [
   accessMethod: ocsp
   accessLocation: URIName: http://wosign-ovca.ocsp-certum.com
,
   accessMethod: caIssuers
   accessLocation: URIName: http://repository.certum.pl/wosign-ovca.cer
]
]

[2]: ObjectId: 2.5.29.35 Criticality=false
AuthorityKeyIdentifier [
KeyIdentifier [
0000: A1 13 54 DC 56 73 2C 27   82 CA C8 84 EF EE BF 00  ..T.Vs,'........
0010: FD 5F AB 56                                        ._.V
]
]

[3]: ObjectId: 2.5.29.19 Criticality=true
BasicConstraints:[
  CA:false
  PathLen: undefined
]

[4]: ObjectId: 2.5.29.31 Criticality=false
CRLDistributionPoints [
  [DistributionPoint:
     [URIName: http://wosign.crl.certum.pl/wosign-ovca.crl]
]]

[5]: ObjectId: 2.5.29.32 Criticality=false
CertificatePolicies [
  [CertificatePolicyId: [2.23.140.1.2.2]
[]  ]
  [CertificatePolicyId: [1.2.616.1.113527.2.5.1.12.2]
[PolicyQualifierInfo: [
  qualifierID: 1.3.6.1.5.5.7.2.2
  qualifier:
0000: 30 81 E4 30 1F 16 18 41   73 73 65 63 6F 20 44 61  0..0...Asseco Da
0010: 74 61 20 53 79 73 74 65   6D 73 20 53 2E 41 2E 30  ta Systems S.A.0
...

]]  ]
]

[6]: ObjectId: 2.5.29.37 Criticality=false
ExtendedKeyUsages [
  serverAuth
  clientAuth
]

[7]: ObjectId: 2.5.29.15 Criticality=true
KeyUsage [
  DigitalSignature
  Key_Encipherment
]

[8]: ObjectId: 2.5.29.17 Criticality=false
SubjectAlternativeName [
  DNSName: *.dcpc.com
  DNSName: dcpc.com
]

[9]: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: B3 A6 53 AD 96 D8 B7 51   09 D5 E1 07 E5 B2 BC E6  ..S....Q........
0010: B5 F2 82 2B                                        ...+
]
]

]
  Algorithm: [SHA256withRSA]
  Signature:
0000: 9A 38 A4 8F B2 A8 CE 9A   B6 98 2F 8F 18 AA 45 79  .8......../...Ey
0010: DE D6 31 22 99 6E D7 14   45 5E C1 1F 06 EC 7F C9  ..1".n..E^......
...

]
[read] MD5 and SHA1 hashes:  len = 1492
0000: 0B 00 05 D0 00 05 CD 00   05 CA 30 82 05 C6 30 82  ..........0...0.
0010: 04 AE A0 03 02 01 02 02   10 36 3F A1 7D 5C 46 08  .........6?..\F.
...

*** ServerHelloDone
[read] MD5 and SHA1 hashes:  len = 4
0000: 0E 00 00 00                                        ....
*** ClientKeyExchange, RSA PreMasterSecret, TLSv1
[write] MD5 and SHA1 hashes:  len = 262
0000: 10 00 01 02 01 00 85 BD   72 17 92 EA 05 0F 0A BD  ........r.......
0010: 90 92 72 51 08 0F 6E 29   C4 A2 F0 D2 BA 78 0B A0  ..rQ..n).....x..
...

DefaultQuartzScheduler_Worker-3, WRITE: TLSv1 Handshake, length = 262
[Raw write]: length = 267
0000: 16 03 01 01 06 10 00 01   02 01 00 85 BD 72 17 92  .............r..
0010: EA 05 0F 0A BD 90 92 72   51 08 0F 6E 29 C4 A2 F0  .......rQ..n)...
...
SESSION KEYGEN:
PreMaster Secret:
0000: 03 01 80 92 4B 03 FA 1D   39 CE F4 82 3D 4A 96 A7  ....K...9...=J..
...
CONNECTION KEYGEN:
Client Nonce:
0000: 5A 72 D8 54 5F 78 84 D8   45 22 CD 49 85 1E F0 B2  Zr.T_x..E".I....
...
Server Nonce:
0000: 5A 72 63 4E F2 CF 4B 0F   CE 58 0E EF 36 D4 F3 3E  ZrcN..K..X..6..>
...
Master Secret:
0000: 93 05 C4 97 B7 FB 23 5F   11 15 8A 8E 2C C1 C7 72  ......#_....,..r
...
Client MAC write Secret:
0000: 22 84 BA CD 61 27 D9 62   5F 84 FA 7A DF 05 DE 0A  "...a'.b_..z....
...
Server MAC write Secret:
0000: D2 AC 73 AC CE F2 D2 00   31 E6 4F 8A DE E3 97 8D  ..s.....1.O.....
0010: 73 52 BA D2                                        sR..
Client write key:
0000: CA FB B2 02 E7 3F 79 93   55 61 CB AB FF EF 1E 76  .....?y.Ua.....v
0010: 10 4C DD DE 83 55 99 6F   28 8E 19 5D 18 25 33 A2  .L...U.o(..].%3.
Server write key:
0000: 1A 25 31 69 D2 3B 45 D7   70 09 5F 9A FF D5 B3 71  .%1i.;E.p._....q
0010: 73 9D D6 52 F0 10 21 8B   8E 13 97 BE B1 02 75 46  s..R..!.......uF
Client write IV:
0000: 68 49 14 C3 A2 97 75 A0   F3 AC 89 DA 31 84 C3 EF  hI....u.....1...
Server write IV:
0000: 2C 03 9D 28 6F C2 7D BD   37 B1 AA E5 30 6E 09 EA  ,..(o...7...0n..
DefaultQuartzScheduler_Worker-3, WRITE: TLSv1 Change Cipher Spec, length = 1
[Raw write]: length = 6
0000: 14 03 01 00 01 01                                  ......
*** Finished
verify_data:  { 56, 169, 160, 43, 198, 100, 229, 140, 160, 251, 76, 146 }
***
[write] MD5 and SHA1 hashes:  len = 16
0000: 14 00 00 0C 38 A9 A0 2B   C6 64 E5 8C A0 FB 4C 92  ....8..+.d....L.
Padded plaintext before ENCRYPTION:  len = 48
0000: 14 00 00 0C 38 A9 A0 2B   C6 64 E5 8C A0 FB 4C 92  ....8..+.d....L.
0010: F5 B1 BB DE F4 25 ED A2   3A 18 9D C8 8E EE 2B A6  .....%..:.....+.
0020: 4A 1E 3A C7 0B 0B 0B 0B   0B 0B 0B 0B 0B 0B 0B 0B  J.:.............
DefaultQuartzScheduler_Worker-3, WRITE: TLSv1 Handshake, length = 48
[Raw write]: length = 53
0000: 16 03 01 00 30 4E 92 1F   3A 1F 91 C8 1B 9F 6B 3A  ....0N..:.....k:
0010: CC 2C 67 2C 9D CA 58 DA   4F 91 21 01 8E D8 5A 37  .,g,..X.O.!...Z7
0020: 67 B8 BD A9 E4 F3 79 E9   9C E6 E1 52 BB 8D 39 0E  g.....y....R..9.
0030: 45 5F F5 7E 7A                                     E_..z
[Raw read]: length = 5
0000: 14 03 01 00 01                                     .....
[Raw read]: length = 1
0000: 01                                                 .
DefaultQuartzScheduler_Worker-3, READ: TLSv1 Change Cipher Spec, length = 1
[Raw read]: length = 5
0000: 16 03 01 00 30                                     ....0
[Raw read]: length = 48
0000: A9 B4 AB 6C 68 BA 47 DB   CF 6E EF 8A F2 6F BD 73  ...lh.G..n...o.s
0010: E4 44 55 C8 93 1B 4B C6   D9 A3 DB 4D 03 3B E0 8D  .DU...K....M.;..
0020: 67 ED E8 35 CA 63 D8 F8   C8 11 40 1B 54 DD 6F F2  g..5.c....@.T.o.
DefaultQuartzScheduler_Worker-3, READ: TLSv1 Handshake, length = 48
Padded plaintext after DECRYPTION:  len = 48
0000: 14 00 00 0C 58 5A F4 F0   FA C9 BE 1D 27 6A A5 3A  ....XZ......'j.:
0010: DC 90 D1 52 C8 53 16 DD   CB 04 E9 F0 F7 46 E2 9F  ...R.S.......F..
0020: 9E A6 EF 63 0B 0B 0B 0B   0B 0B 0B 0B 0B 0B 0B 0B  ...c............
*** Finished
verify_data:  { 88, 90, 244, 240, 250, 201, 190, 29, 39, 106, 165, 58 }
***
%% Cached client session: [Session-8, TLS_RSA_WITH_AES_256_CBC_SHA]
[read] MD5 and SHA1 hashes:  len = 16
0000: 14 00 00 0C 58 5A F4 F0   FA C9 BE 1D 27 6A A5 3A  ....XZ......'j.:
DefaultQuartzScheduler_Worker-3, setSoTimeout(9000000) called
Padded plaintext before ENCRYPTION:  len = 960
##发送数据
0000: 50 4F 53 54 20 2F 69 64   6D 2F 77 73 2F 55 6E 69  POST /idm/ws/Uni
0010: 74 53 65 72 76 69 63 65   20 48 54 54 50 2F 31 2E  tService HTTP/1.
0020: 30 0D 0A 43 6F 6E 74 65   6E 74 2D 54 79 70 65 3A  0..Content-Type:
0030: 20 74 65 78 74 2F 78 6D   6C 3B 20 63 68 61 72 73   text/xml; chars
0040: 65 74 3D 75 74 66 2D 38   0D 0A 41 63 63 65 70 74  et=utf-8..Accept
0050: 3A 20 61 70 70 6C 69 63   61 74 69 6F 6E 2F 73 6F  : application/so
0060: 61 70 2B 78 6D 6C 2C 20   61 70 70 6C 69 63 61 74  ap+xml, applicat
0070: 69 6F 6E 2F 64 69 6D 65   2C 20 6D 75 6C 74 69 70  ion/dime, multip
0080: 61 72 74 2F 72 65 6C 61   74 65 64 2C 20 74 65 78  art/related, tex
0090: 74 2F 2A 0D 0A 55 73 65   72 2D 41 67 65 6E 74 3A  t/*..User-Agent:
00A0: 20 41 78 69 73 2F 31 2E   34 0D 0A 48 6F 73 74 3A   Axis/1.4..Host:
00B0: 20 69 64 6D 74 65 73 74   2E 64 63 70 63 2E 63 6F   ***.***.co
00C0: 6D 3A 38 34 34 33 0D 0A   43 61 63 68 65 2D 43 6F  m:8443..Cache-Co
...

DefaultQuartzScheduler_Worker-3, WRITE: TLSv1 Application Data, length = 960
[Raw write]: length = 965
0000: 17 03 01 03 C0 76 73 C9   5C 37 18 B3 8D 85 B6 4A  .....vs.\7.....J
0010: A5 FF 06 E0 81 74 E2 46   22 C7 32 12 51 A6 C4 10  .....t.F".2.Q...
0020: 7E 1D 14 C6 1E 1B 44 95   57 6E AC 83 AA 8D A7 92  ......D.Wn......
0030: 4C 34 01 5D E9 A1 1E 8C   BB 03 55 D9 7A 09 0F 1E  L4.]......U.z...
0040: 5D C4 71 46 CA 29 72 0D   C1 CC F8 CE 6E F9 51 3A  ].qF.)r.....n.Q:
...

[Raw read]: length = 5
0000: 17 03 01 01 60                                     ....`
[Raw read]: length = 352
0000: CD C2 06 A2 9F EA 9F 00   C7 27 5E F0 B3 C4 3D 78  .........'^...=x
0010: 2E 9B F4 52 08 FF D7 05   2E A8 1B 42 3F 9A 3D 73  ...R.......B?.=s
...
DefaultQuartzScheduler_Worker-3, READ: TLSv1 Application Data, length = 352
##接收数据
Padded plaintext after DECRYPTION:  len = 352
0000: 48 54 54 50 2F 31 2E 31   20 32 30 30 20 0D 0A 43  HTTP/1.1 200 ..C
0010: 6F 6E 74 65 6E 74 2D 54   79 70 65 3A 20 74 65 78  ontent-Type: tex
0020: 74 2F 78 6D 6C 3B 63 68   61 72 73 65 74 3D 55 54  t/xml;charset=UT
0030: 46 2D 38 0D 0A 44 61 74   65 3A 20 54 68 75 2C 20  F-8..Date: Thu,
0040: 30 31 20 46 65 62 20 32   30 31 38 20 30 39 3A 30  01 Feb 2018 09:0
0050: 35 3A 32 35 20 47 4D 54   0D 0A 43 6F 6E 6E 65 63  5:25 GMT..Connec
0060: 74 69 6F 6E 3A 20 63 6C   6F 73 65 0D 0A 0D 0A 3C  tion: close....<
0070: 73 6F 61 70 3A 45 6E 76   65 6C 6F 70 65 20 78 6D  soap:Envelope xm
0080: 6C 6E 73 3A 73 6F 61 70   3D 22 68 74 74 70 3A 2F  lns:soap="http:/
0090: 2F 73 63 68 65 6D 61 73   2E 78 6D 6C 73 6F 61 70  /schemas.xmlsoap
00A0: 2E 6F 72 67 2F 73 6F 61   70 2F 65 6E 76 65 6C 6F  .org/soap/envelo
...
[Raw read]: length = 5
0000: 15 03 01 00 20                                     ....
[Raw read]: length = 32
0000: 10 E4 3E EF D9 D6 A2 0F   F3 11 85 6F 8F 68 48 C8  ..>........o.hH.
0010: 26 73 2E C8 FA 9D 00 03   AB 3B 79 46 DE ED CC 40  &s.......;yF...@
DefaultQuartzScheduler_Worker-3, READ: TLSv1 Alert, length = 32
Padded plaintext after DECRYPTION:  len = 32
0000: 01 00 C2 40 0A 51 E1 F1   28 06 65 73 E4 84 81 01  ...@.Q..(.es....
0010: 8F A9 A1 FD A9 6E 09 09   09 09 09 09 09 09 09 09  .....n..........
DefaultQuartzScheduler_Worker-3, RECV TLSv1 ALERT:  warning, close_notify
DefaultQuartzScheduler_Worker-3, called closeInternal(false)
DefaultQuartzScheduler_Worker-3, SEND TLSv1 ALERT:  warning, description = close_notify
Padded plaintext before ENCRYPTION:  len = 32
0000: 01 00 4F BB 75 8C 9C 8A   12 64 D7 62 F6 AF 81 90  ..O.u....d.b....
0010: 92 0E 91 25 6F 9B 09 09   09 09 09 09 09 09 09 09  ...%o...........
DefaultQuartzScheduler_Worker-3, WRITE: TLSv1 Alert, length = 32
[Raw write]: length = 37
0000: 15 03 01 00 20 C1 60 66   8A 2A 1D A5 6C 82 C3 59  .... .`f.*..l..Y
0010: A8 D5 E9 28 70 1D 93 F1   DD 34 00 14 1F 56 12 46  ...(p....4...V.F
0020: 33 0C 4C 3F 51                                     3.L?Q
DefaultQuartzScheduler_Worker-3, called closeSocket(selfInitiated)
DefaultQuartzScheduler_Worker-3, called close()
DefaultQuartzScheduler_Worker-3, called closeInternal(true)
DefaultQuartzScheduler_Worker-3, called close()
DefaultQuartzScheduler_Worker-3, called closeInternal(true)
```



### 证书识别

Java程序对受信证书的查找顺序是：

1. javax.net.ssl.trustStore

   就是System.setProperty("javax.net.ssl.trustStore","D:\\cacert.keystore");或者通过JVM参数指定：-Djavax.net.ssl.trustStore=D:\cacert.keystore。

2. JAVA_HOME\jre\lib\security\jssecacerts 【推荐】

   默认没有jssecacerts，一般把cacerts复制成jssecacerts，然后再使用keytool来修改它。jsse的全称是Java Secure Socket Extension。

3. JAVA_HOME\jre\lib\security\cacerts

   通常情况下不就建议通过-Djavax.net.ssl.trustStore=D:\cacert.keystore来直接修改keystore文件的位置，因为这样会造成系统中其他模块可能找不到自己对应的证书了。推荐使用jssecacerts。我们需要什么证书直接往这个位置导就行了。程序不需要做调整。

### Springboot 程序实现

我们希望在Springboot应用每次启动后都向服务器请求证书，并将证书更新到Keystore中

```
@ConditionalOnProperty(value = "cert.installation", havingValue = "true")
@ConfigurationProperties("cert")
@Configuration
public class InstallCert implements CommandLineRunner {

	private static Logger logger = LoggerFactory.getLogger(InstallCert.class);

	/**
	 * 生成的证书需要存放的security路径，即： %JAVA_HOME%\jre\lib\security
	 */
	private String path;

	/**
	 * 需要SSL认证的域名
	 */
	private List<String> domains = new ArrayList<>();


	private String password = "changeit";

	public String getPath() {
		return path;
	}

	public void setPath(String path) {
		this.path = path;
	}

	public List<String> getDomains() {
		return domains;
	}

	public void setDomains(List<String> domains) {
		this.domains = domains;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(String password) {
		this.password = password;
	}

	@Override
	public void run(String... strings) throws Exception {
		logger.info("ready to install valid certs");
		if (!Collections.isEmpty(domains)){
			logger.info("domains is {}", domains.toString());
			String[] arrayStr = new String[domains.size()];
			installValidCerts(domains.toArray(arrayStr));
		}
	}


	private void installValidCerts(String[] args) throws Exception {
		logger.info("Usage: java InstallCert <host>[:port] [passphrase]");

		char[] passphrase = password.toCharArray();

		String certPath = path;
		if (Strings.isNullOrEmpty(path)){
			char separator = File.separatorChar;
			certPath = System.getProperty("java.home") + separator + "lib" + separator + "security";
		}
		File dir = new File(certPath);

//		File file = new File("jssecacerts");
//		if (file.isFile() == false) {
//			char SEP = File.separatorChar;
//			File dir = new File(System.getProperty("java.home") + SEP + "lib" + SEP + "security");
//			file = new File(dir, "jssecacerts");
//			if (file.isFile() == false) {
//				file = new File(dir, "cacerts");
//			}



		KeyStore ks = KeyStore.getInstance(KeyStore.getDefaultType());
		InputStream in = null;
		logger.info("jre security dir path: {}", dir.getCanonicalPath());
		File file = new File(dir, "jssecacerts");
		if (!file.exists() && !file.isFile()){
			file.createNewFile();
			File file1 = new File(dir, "cacerts");
			if (!file1.isFile()){
				logger.info("no certs will be registered, casue no cacerts file was found");
				return;
			}
			in = new FileInputStream(file1);
			ks.load(in, passphrase);
			logger.info("file jssecacerts is created successfully");
		}else {
			in = new FileInputStream(file);
			ks.load(in, passphrase);
			logger.info("file jssecacerts is existing, will not create again");
		}
		IOUtils.closeQuietly(in);

		logger.info("Loading KeyStore " + file + "...");


		SSLContext context = SSLContext.getInstance("TLS");
		TrustManagerFactory tmf = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
		tmf.init(ks);
		X509TrustManager defaultTrustManager = (X509TrustManager) tmf.getTrustManagers()[0];
		SavingTrustManager tm = new SavingTrustManager(defaultTrustManager);
		context.init(null, new TrustManager[] { tm }, null);
		SSLSocketFactory factory = context.getSocketFactory();


		OutputStream out = new FileOutputStream(file);
		for (String domain: args){
			logger.info("domain is {}", domain);
			String[] c = args[0].split(":");
			String host = c[0];
			int port = (c.length == 1) ? 443 : Integer.parseInt(c[1]);
			logger.info("Opening connection to " + host + ":" + port + "...");
			SSLSocket socket = (SSLSocket) factory.createSocket(host, port);
			socket.setSoTimeout(10000);
			try {
				logger.info("Starting SSL handshake...");
				socket.startHandshake();
				socket.close();
				logger.info("No errors, certificate is already trusted");
			} catch (SSLException e) {
				e.printStackTrace(System.out);
			}

			X509Certificate[] chain = tm.chain;
			if (chain == null) {
				logger.info("Could not obtain server certificate chain, continue next one");
				continue;
			}


			logger.info("Server sent " + chain.length + " certificate(s):");
			MessageDigest sha1 = MessageDigest.getInstance("SHA1");
			MessageDigest md5 = MessageDigest.getInstance("MD5");
			for (int i = 0; i < chain.length; i++) {
				X509Certificate cert = chain[i];
				logger.info(" " + (i + 1) + " Subject " + cert.getSubjectDN());
				logger.info("   Issuer  " + cert.getIssuerDN());
				sha1.update(cert.getEncoded());
				logger.info("   sha1    " + toHexString(sha1.digest()));
				md5.update(cert.getEncoded());
				logger.info("   md5     " + toHexString(md5.digest()));
			}

			int k=0;

			X509Certificate cert = chain[k];
			String alias = host + "-" + (k + 1);
			ks.setCertificateEntry(alias, cert);
			ks.store(out, passphrase);

			logger.info(cert.toString());
			logger.info("Added {} certificate to keystore 'jssecacerts' using alias {}", domain, alias);
		}
		IOUtils.closeQuietly(out);
	}

	private static final char[] HEXDIGITS = "0123456789abcdef".toCharArray();

	private static String toHexString(byte[] bytes) {
		StringBuilder sb = new StringBuilder(bytes.length * 3);
		for (int b : bytes) {
			b &= 0xff;
			sb.append(HEXDIGITS[b >> 4]);
			sb.append(HEXDIGITS[b & 15]);
			sb.append(' ');
		}
		return sb.toString();
	}



	private static class SavingTrustManager implements X509TrustManager {

		private final X509TrustManager tm;
		private X509Certificate[] chain;

		SavingTrustManager(X509TrustManager tm) {
			this.tm = tm;
		}

		@Override
		public X509Certificate[] getAcceptedIssuers() {
			throw new UnsupportedOperationException();
		}

		@Override
		public void checkClientTrusted(X509Certificate[] chain, String authType) throws CertificateException {
			throw new UnsupportedOperationException();
		}

		@Override
		public void checkServerTrusted(X509Certificate[] chain, String authType) throws CertificateException {
			this.chain = chain;
			tm.checkServerTrusted(chain, authType);
		}
	}
}
```

接下来我们只需要在YAML（applicaton.yml）中配置要申请握手的服务器就好

```yaml
cert:
  installation: true
  domains:
    - bidxxxxxxx.xxxxxxxxxxxxxx.com
    - suppxxxxxx.xxxxx.com
```

至此，就不会出现证书找不到的问题了



### 参考

1. https://www.cnblogs.com/huqiaoblog/p/8398009.html
2. http://www.ruanyifeng.com/blog/2014/09/illustration-ssl.html
3. https://blog.csdn.net/catoop/article/details/80819638
