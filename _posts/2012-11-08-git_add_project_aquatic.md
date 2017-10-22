---
layout: post
title: "博客主题做成了主题包，可以直接pull下来"
description: ""
category: IT
analytics: true
tags: [IT, aquatic, theme, jekyll, github]
yourblogpath: "~/yourblog"
---
在github上添加了个[jekyll 的主题 aquatic](http://ericwang.github.com/aquatic)

在`{{page.yourblogpath}}`目录下，使用命令来下载
{% highlight ruby %}
rake theme:install git="https://github.com/ericwang/aquatic.git" 
{% endhighlight %}
此命令会将主题文件下载到`{{page.yourblogpath}}/_theme_packages`目录下。
最后会提示是否切换到新主题，可以直接切换。如果不想切换，可以使用下面的命令切换。
{% highlight bash %}
rake theme:switch name="aquatic"
{% endhighlight %}
主题安装后，输入一下命令
{% highlight bash %}
jekyll --server
{% endhighlight %}
在浏览器中输入`http://localhost:4000/`就可以预览了。

不喜欢可以再切换回去，主题名字就是`{{page.yourblogpath}}/_includes/themes`里面的目录名。
可以使用上面的命令随时切换。也可以手工切换，在`_layout/default.html`中，修改name后面的值。
{% highlight bash %}
---
theme :
  name : theme_name
---
{% endhighlight %}
删除很简单，只要删除`{{page.yourblogpath}}/_includes/themes`里面的`aquatic`目录，和`{{page.yourblogpath}}/assets/themes`里的`aquatic`目录即可。
