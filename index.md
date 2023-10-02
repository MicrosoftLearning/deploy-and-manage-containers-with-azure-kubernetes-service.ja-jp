---
title: オンラインでホスティングされている手順
permalink: index.html
layout: home
---

# コンテンツ ディレクトリ

各ラボの演習とデモへのハイパーリンクを以下に示します。

## ラボ

{% assign Exercise = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| モジュール| 演習 |
| --- | --- | 
{% for activity in Exercise  %} | {{ activity.Exercise.module }} | [{{ activity.Exercise.title }}{% if activity.Exercise.type %} - {{ activity.Exercise.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

