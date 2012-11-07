---
layout: page
title: (◕‿‿◕)要相信你的CPU!
tagline:
---
{% include JB/setup %}
<ul class="posts">
    {% for post in site.posts %}
    <li><p class="date" cate="{{ post.categories }}">{{ post.date | date:"%Y-%m-%d" }}</p> <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
</ul>
