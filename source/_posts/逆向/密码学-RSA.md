---
title: 密码学_RSA
date: 2021-04-10 15:25:44
tags:
    - RSA
categories:
    - 逆向
---

# 1. 密码学
## 1.1 密码学概述
密码学是指研究信息加密，破解密码的技术科学。密码学的起源可追溯到2000年前。而当今的密码学是以数学为基础的。

## 1.2 发展
1. 在1976年以前，所有的加密方法都是同一种模式：加密、解密使用同一种算法。在交互数据的时候，彼此通信的双方就必须将规则告诉对方，否则没法解密。那么加密和解密的规则（简称密钥），它保护就显得尤其重要。传递密钥就成为了最大的隐患。这种加密方式被成为对称加密算法（symmetric encryption algorithm）
2. 1976年，两位美国计算机学家 迪菲（W.Diffie）、赫尔曼（ M.Hellman ） 提出了一种崭新构思，可以在不直接传递密钥的情况下，完成密钥交换。这被称为“<B>迪菲赫尔曼密钥交换</B>”算法。开创了密码学研究的新方向。
3. 1977年三位麻省理工学院的数学家 <B>罗纳德·李维斯特（Ron Rivest）、阿迪·萨莫尔（Adi Shamir）和伦纳德·阿德曼（Leonard Adleman）</B>一起设计了一种算法，可以实现<B>非对称加密</B>。这个算法用他们三个人的名字命名，叫做<B>RSA算法</B>。

# 2. RSA
RSA加密方式比较特殊，需要两个密钥：公开密钥简称公钥（publickey）和私有密钥简称私钥（privatekey）。公钥加密，私钥解密；私钥加密，公钥解密。这个加密算法就是伟大的RSA。

## 2.1 欧拉函数

首先考虑什么是离散对数：在整数中，离散对数（英语：Discrete logarithm）是一种基于同余运算和[原根](https://baike.baidu.com/item/%E5%8E%9F%E6%A0%B9)的一种对数运算。

互质关系：如果两个正整数，除了1以外，没有其他公因数，我们就称这两个数是互质关系（coprime）。

有一个正整数n，请问在小于等于n的正整数之中，有多少个与n构成互质关系？
计算这个值的方式就是<b>欧拉函数</b>。用φ(n)来表示。φ -> phi

例如：
> 计算8的欧拉函数，和8互质的数有1、3、5、7
> φ(8) = 4
> 
> 例如计算7的欧拉函数，和7互质的数有1、2、3、4、5、6
> φ(7) = 6
> 
> 例如计算56的欧拉函数
> φ(56) = φ(8) * φ(7) = 4 * 6 = 24

### 2.2.1 欧拉函数的特性：
1. 当n是质数的时候，φ(n)=n-1。
2. 如果n可以分解成两个互质的整数之积，如n=A*B则：φ(A*B)=φ(A)* φ(B)

根据以上两点得到：
如果N是两个质数P1 和 P2的乘积则φ(N)=φ(P1)* φ(P2)=(P1-1)*(P2-1)


## 2.2 欧拉定理
如果两个正整数m和n互质，那么m的φ(n)次方减去1，可以被n整除。

m^φ(n) % n = 1

也就是说，m的φ(n)次方减去1，能被n整出。

例如：
```
// m = 3， n = 11, φ(n) = 10
// 再Python3的环境下，快速输出结果，如下

>>> 3**10%11
1
```

### 2.2.1 费马小定理

欧拉定理的特殊情况：如果两个正整数m和n互质，而且n为质数！那么φ(n)结果就是n-1。

m^(n-1) % n = 1

这就是一开始所说的那种情况，如果n为质数，则φ(n)= n-1

## 2.3 模反元素
如果两个正整数e和x互质，那么一定可以找到整数d，使得 ed-1 被x整除。那么d就是e对于x的“模反元素”。
e*d % x = 1

## 2.4 公式推导过程：

1. 1^k = 1, 1的k次方等于1

2. 1 * m = m

3. 根据欧拉定理：m^φ(n) % n = 1，公式两边分别执行k次方。

4. (m^φ(n) % n)^k = 1^k 公式简化后：

5. m^k*φ(n) % n = 1，等式两边分别乘以m

6. (m^k*φ(n) % n) * m = 1 * m 简化后：

7. m^k*φ(n)+1 % n = m

8. 根据模反元素的公式：e*d % x = 1，则 e*d = k*x + 1

到这里，有没有发现什么？看推导的第7步和第8步的公式，k*φ(n)+1 和 k*x+1是不是很相似。则 e * d = k*φ(n)+1

那最后是不是可以得出：m^ed % n = m

这就是大名鼎鼎的<b>迪菲赫尔曼密钥交换</b>。
我们假设 m = 3，n = 17，
服务器生成随机数 e = 15，客户端生成随机数 d = 13

服务器 m^e % n，结果是 3^15 % 17 = 6， 把明文6传给客户端。
客户端 m^d % n，结果是 3^13 % 17 = 12， 把明文12传给服务器。

两端拿到的数据分别进行解密：
客户端 6^13 % 17 = 10
服务器 12^15 % 17 = 10

<b>迪菲赫尔曼密钥交换</b>:
3^15*12 % n = 3^13*15 % n
m^e*d % n = 10 = m^d*e % n

从这个时候开始，开创了密码学研究的新方向，也为RSA加密打下了基础。RSA对上面的公式进行了拆分：

m^e % n = c
c^d % n = m

这就是RSA诞生的原理。其中
m^e % n = c 进行加密
c^d % n = m 进行解密

n、e是公钥，n、d是私钥，m是明文，c是密文。

验证： m = 4， n = 15， φ(n) = 8， e = 3， 
求出d（d是e对于φ(n)的“模反元素”） d = 11，19

```
// python3环境
>>> 4**33%15
4

// 加入m = 5
>>> 5**33%15
5

// 加入m = 15
>>> 15**33%15
0
```

到此，就看到了整个的RAS的基本过程。

# 3. RSA总结
## 3.1 说明
1. n会非常大，长度一般为1024个二进制位。（目前人类已经分解的最大整数，232个十进制位，768个二进制位）
2. 由于需要求出φ(n)，所以根据欧函数特点，最简单的方式n 由两个质数相乘得到: 质数：p1、p2，Φ(n) = (p1 -1) * (p2 - 1)
3. 最终由φ(n)得到e 和 d。总共生成6个数字：p1、p2、n、φ(n)、e、d

## 3.2 关于RSA的安全：

除了公钥用到了n和e 其余的4个数字是不公开的。目前破解RSA得到d的方式如下：
1. 要想求出私钥 d。由于e*d = φ(n)*k + 1。要知道e和φ(n);
2. e是知道的，但是要得到 φ(n)，必须知道p1 和 p2。
3. 由于 n=p1*p2。只有将n因数分解才能算出。

## 3.3 RSA特点
1. 相对来说比较安全（非对称加密）
2. 效率低 
3. 加密小数据，一般加密核心数据


# 4. 终端演示
由于Mac系统内置OpenSSL(开源加密库)，所以我们可以直接在终端上使用命令来玩RSA。OpenSSL中RSA算法常用指令主要有三个：

| 命令 | 含义 |
| :-- | :-- |
|genrsa|生成并输入一个RSA密钥|
|rsatul|使用RSA密钥进行加密、解密、签名和验证等运算|
|rsa|处理RSA密钥的格式转化等问题|

```
// 使用命令行演示
// 1. 生成RSA密钥，密钥长度为1024bit
> openssl genrsa -out private.pem 1024

// 2. 从私钥中提取公钥
> openssl rsa -in private.pem -pubout -out public.pem

// 3. 将私钥转换为明文信息
> openssl rsa -in private.pem -text -out private.txt

// 4. 通过公钥加密 首先创建一个message.txt文件，并输入内容
> vi message.txt 
> openssl rsautl -encrypt -in message.txt -inkey public.pem -pubin -out enc.txt

// 5. 通过私钥解密
> openssl rsautl -decrypt -in enc.txt -inkey private.pem -out dec.txt

// 6. 通过私钥加密数据
> openssl rsautl -sign -in message.txt -inkey private.pem -out penc.txt
// 7. 公钥解密数据
> openssl rsautl -verify -in penc.txt -inkey public.pem -pubin -out pdec.txt

```

# 5. 代码演示
由于Xcode是没有办法直接使用pem文件的，所以需要转化，首先需要生成请求的cer文件

```
> openssl req -new -key private.pem -out rsacert.csr
-----
> Country Name (2 letter code) []:CN
> State or Province Name (full name) []:Beijing
> Locality Name (eg, city) []:Chaoyang
> Organization Name (eg, company) []:Wangjing
> Organizational Unit Name (eg, section) []:alan.com
> Common Name (eg, fully qualified host name) []:alan.com
> Email Address []:alan@163.com

> Please enter the following 'extra' attributes
> to be sent with your certificate request
// 这里不使用密码，其他根据要求自己填写。
> A challenge password []:
```

拿到csr文件之后，需要去请求签名文件，一般用于服务器，下发证书，比如https

```
> openssl x509 -req -days 3650 -in rsacert.csr -signkey private.pem -out rsacert.crt
```

然后我们通过crt证书生成Xcode可用的p12证书。

```
> openssl pkcs12 -export -out p.p12 -inkey private.pem -in rsacert.crt
// 注意这里需要输入密码，123456
```

在iOS系统钟，ctr文件是无法直接使用的，所以还需要把ctr文件转化为der文件。

```
> openssl x509 -outform der -in rsacert.crt -out rsacert.der
```

生成的rsacert.der和p.p12文件就相当于我们要使用的公钥和私钥。代码就先不放了哈~

感谢~



