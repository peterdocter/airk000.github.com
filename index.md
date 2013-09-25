---
layout: page
title:
tagline:
---
{% include JB/setup %}

*不会写HTML CSS好苦恼Orz！！！*

##Posts
<div id="home">
  <ul class="posts">
    {% for post in site.posts %}
      <li>
        <span class="post_date">{{ post.date | date_to_string }}</span> &raquo; <a href="{{ post.url }}">{{ post.title }}</a>
        <p>{{ post.summary }}</p>
      </li>
    {% endfor %}
  </ul>
</div>

##About ME

    E-mail: airk908@gmail.com
    Github: github.com/airk000
    Twitter\Weibo\Google+ All airk000
