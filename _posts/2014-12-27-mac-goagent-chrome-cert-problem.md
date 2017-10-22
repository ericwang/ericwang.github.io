---
layout: post
title: mac osx 使用goagent，在chrome下访问google，不是私密连接的问题
description: ""
category: IT
analytics: true
tags: [mac, goagent, 2014, 十二月, 27日]
---

### 解决办法

确认已经安装好goagent

确认 chrome 39

访问 google 网站会出现提示 不是私密连接 的问题。

解决办法:
1.删除goagent local目录下的 CA.crt 文件，和 certs 文件夹。
2.启动goagent，CA.crt文件会从新生成。
3.打开 钥匙串访问.app -&gt; 左侧钥匙串中选择\`系统\` -&gt; 文件 -&gt; 导入项目 -&gt; 选择刚刚生成的CA.crt -&gt;全部信任
4.要输入两次管理员密码
5.\`点击导入的证书 -&gt; 右键 显示简介 -&gt; 信任 -&gt; 全部选择 始终信任\`
6.输入管理员密码
7.重启浏览器 完成

第五步是最重要的。导入证书后，必须在简介中，再次设置信任，才能生效。巨坑

2014.12.27
