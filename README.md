## Quarto
https://quarto.org/ 
https://github.com/quarto-dev
https://github.com/quarto-dev/quarto-cli
https://github.com/mcanouil/awesome-quarto

## Migration
https://livefreeordichotomize.com/posts/2022-09-19-migrating-from-hugo-to-quarto/
https://www.andreashandel.com/posts/2022-10-01-hugo-to-quarto-migration/
https://beamilz.com/posts/2022-06-05-creating-a-blog-with-quarto/en/
https://albert-rapp.de/posts/13_quarto_blog_writing_guide/13_quarto_blog_writing_guide.html
https://blog.djnavarro.net/posts/2022-04-20_porting-to-quarto/

## Workflow

``` bash
mkdir posts/migrating-from-hugo-to-quarto
touch posts/migrating-from-hugo-to-quarto/index.qmd
```

``` yaml
---
title: "Migrating my hugo blog to quarto"
subtitle: | 
  I have moved this blog from hugo over to quarto, and taken notes.
date: "2022-11-20"
categories: [quarto, blogging, hugo]
image: "img/preview.jpg"
---
```

``` bash
quarto preview
```

```
quarto publish gh-pages
```
