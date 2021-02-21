---
title: "{{ replace .Name "-" " " | title }}" # Title of the blog post.
date: {{ .Date }} # Date of post creation.
description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: true # Sets whether to render this page. Draft of true will not be rendered.
toc: false
# menu: main
featureImage: "/images/path/file.jpg"
thumbnail: "/images/path/thumbnail.png"
shareImage: "/images/path/share.png"
codeMaxLines: 10
codeLineNumbers: false
figurePositionShow: true
#series: "opengl-tutorials"
categories:
  - Technology
tags:
  - Tag_name1
  - Tag_name2
# comment: false # Disable comment if false.
---

**Insert Lead paragraph here.**