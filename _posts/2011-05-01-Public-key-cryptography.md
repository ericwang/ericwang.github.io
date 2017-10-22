---
layout: post
title: Public-key cryptography
description: ""
category: IT
analytics: true
tags: [pulic, key, cryptographyt, pic]
---

下午和鱼琢磨了半天非对称加密及其作用。
总结了半天，我画了一张图，免得以后忘了。如果有不对的地方，帮忙指正。
公钥，公共的钥匙。任何人都可以拥有她。
所以，用户a，用户b都拥有公钥a和公钥b。
私钥，私有的钥匙。每个人只有自己的钥匙。
所以，用户a有私钥a，用户b有私钥b。
那现在来看看用户a和用户b都各自拥有哪些钥匙。
用户a：公钥a，公钥b，私钥a。
用户b：公钥a，公钥b，私钥b。

非对称加密算法还有一个原则：用公钥加密，使用对应的私钥解密。用私钥加密，使用对应的公钥解密。

遵循这个原则，用户a，可以使用公钥b加密明文内容，生成密文。那么，用户b就可以使用私钥b解密，从而解密密文为明文。
当然实际流程并非这么简单。请看下图…
<img src="/image/Public-key-cryptography.PNG" width="700" title="Public-key-cryptography" />
我也不知道此图对不对。凑合看吧…
从左上角开始解释。
用户a将明文使用哈希算法，生成“数字指纹”，用来验证信息完整性，防止明文在传输过程中被篡改。
数字指纹内容加上“用户a的私钥”，使用非对称加密算法进行加密，得到“用户a的数字签名”。她里面拥有“明文信息摘要”的数据。这个“用户a的数字签名”是为了验证信息来源的可靠性，防止明文被他人假冒发出。
然后把“明文数据”和“用户a的数字签名”还有“用户a的公钥”（用户a数字证书）这些数据，加上“对称密钥”，使用对称加密算法，得到“密文”。“密文”里就拥有“明文”，“用户a的数字签名”，和“a的公钥”这三项数据了。
然后，将刚才加密“明文”数据使用到的“对称密钥”，加上“用户b的公钥”，使用非对称加密算法加密，得到“数字信封”。“数字信封”里就拥有解密密文内容要用到的“对称密钥”的信息了。

将“数字信封”和“密文”数据传送给用户b。

用户b首先将“数字信封”解密，由于“数字信封”使用的是“用户b的公钥”加密，那么用户b只要使用“用户b的私钥”就能解开“数字信封”。
解开“数字信封”后，得到用于加密明文信息的“对称密钥”。使用“对称密钥”，用过对称算法，将“密文”解密。得到“明文”，“用户a的数字签名”，和“用户a的公钥”。这三个数据。
将“用户a的数字签名”，通过“用户a的公钥”，使用非对称算法解密，得到“明文”的“数字指纹”。
然后将解密得到的“明文”信息直接使用哈希算法得到的“数字指纹”信息和刚才用“数字签名”解密出来的“数字指纹”做对比。
如果一致，说明“明文”内容没有被篡改过。并且“密文”数据的来源是可靠的。如果不一致，说明要不就是“明文”数据被篡改过，要不就是“用户a的数字签名”是伪造的（用户a的私钥是伪造的）。