---
title: "그래픽스 문서"
layout: archive
permalink: categories/graphics_doc
author_profile: true
sidebar_main: true
---
<!--permalink, 이 파일의 뒷 이름은 같아야하는 듯--!>
<!-- 공백이 포함되어 있는 카테고리 이름의 경우 site.categories.['a b c'] 이런식으로! -->

{% assign posts = site.categories.GraphicsDoc %}
{% for post in posts %} {% include archive-single3.html type=page.entries_layout %} {% endfor %}
