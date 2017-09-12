---
title: hexo+github搭建自己的微博详解
date: 2017-08-13 17:14:31
tags: 
         - html
         - js
         - java
categories: 
        - test
        - js
layout: true
---
### the first section
  ``````
  this is a  code area haha 
  
  ``````
   <!-- more -->
  {% codeblock _.compact http://underscorejs.org/#compact Underscore.js %}
  _.compact([0, 1, false, 2, '', 3]);
  => [1, 2, 3]

  {% endcodeblock %}
  {% img 你好 /uploads/head23.jpeg 200 200 一张图片 %}
  
  <% for (var link in site.data.menu) { %>
    <a href="<%= site.data.menu[link] %>"> <%= link %> </a>
  <% } %>

