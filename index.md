---
layout: default
title: Kengo's Blog
description: Thoughts and experiments on software engineering
---

{% assign totalPosts = site.posts | size %}
{% assign totalWords = 0 %}

{% for post in site.posts %}
{% assign postWords = post.content | number_of_words %}
{% assign totalWords = totalWords | plus: postWords %}
{% endfor %}

I like building scalable systems by day and trying to reinvent the wheel by night. I was looking for a quiet place on the Internet to document my thougths and experiments on software engineering for some time, and finally decided to start a blog in November 2024.

I live in London and work as a Software Engineer at Isomorphic Labs [^1]. I started my career at a marketplace company called Mercari in Tokyo in 2020, then I came back to London in 2022 and worked for Seldon and Bloomberg before joining the current company.

The minimal design of this website is _heavily_ inspired by [Alex Molas's blog], whose content I also highly recommend.

[Alex Molas's blog]: https://www.alexmolas.com/

---

<h2>Select Posts</h2>

{% for post in site.posts %}
{% if post.feature %}

  <li>
    <span class="post-date">{{ post.date | date: "%b %d %Y" }}</span> · <a href="{{ post.url }}">{{ post.title }}</a>
  </li>
{% endif %}
{% endfor %}

---

[^1]: Opinions expressed in this blog are solely my own and do not express the views of my employer.
