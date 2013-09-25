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
<footer>
    <li>
        <a href="http://github.com/airk000">Github</a>
    </li>
    <li>
        <a href="http://weibo.com/airk000">Weibo</a>
    </li>
    <li>
        <a href="https://plus.google.com/115315602279381976098">Google+</a>
    </li>
    <li>
        <a >E-mail: airk908@gmail.com</a>
    </li>
</footer>
