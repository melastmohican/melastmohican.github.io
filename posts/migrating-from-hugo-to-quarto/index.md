---
title: "Migrating my Hugo blog to Quarto"
subtitle: | 
  I have moved this blog from Hugo over to Quarto, and taken notes.
date: "2022-11-20"
categories: [quarto, blogging, hugo]
image: "preview.jpg"
---

I haven't been writing any post for a while and tried it again recently. It looked like I forgot what my workflow is. Hugo is supposed to make things easy but maybe because my setup was slightly different than the one they recommend now I struggled to get up to speed.

## My Hugo workflow

1. I am using separate repo and have Hugo public folder setup as git submodule to actual Github pages repo.
2. Theme is is also submodule but it does not matter for deployment

``` bash
.
├── README.md
├── archetypes
│   └── default.md
├── config.toml
├── content
│   ├── about.md
│   └── posts
│       ├── murphys-laws.md
│       ├── rusty_feather.md
│       └── terraform_best_practice_guide_for_developers.md
├── deploy.ps1
├── deploy.sh
├── public
│   ├── 404.html
├── resources
├── static
│   └── images
│       ├── avatar.png
│       └── esp-rust-board.gif
└── themes
    └── hugo-coder
```

1. I usually use new command to create new post:  `hugo new posts/rusty_feather.md`
2. I can test it locally: `hugo server -D`
3. Finally I got deployment scripts for Mac and Windows.

``` bash
#!/bin/sh

# If a command fails then the deploy stops
set -e

printf "\033[0;32mDeploying updates to GitHub...\033[0m\n"

# Build the project.
hugo -t hugo-coder

# Go To Public folder
cd public

# Add changes to git.
git add .

# Commit changes.
msg="rebuilding site $(date)"
if [ -n "$*" ]; then
	msg="$*"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin master
```
>I could get it converted to Github Action which is current recommendation for deploying your pages.


Pretty straightforward if I just use plain Markdown but in my recent post I decided to put an images in and I forgot where these things are supposed to be be located. It could be `static/images` or `resource/images` and nothing seemed to work for me. I got even more frustrated when I wanted refer to image from internet. Standard Markdown did not work, why I need shortcode to include image?
```
{{< figure src="https://raw.githubusercontent.com/esp-rs/esp-rust-board/master/assets/rust_board_v1.2_diagram.png" title="ESP32-C3-DevKit-RUST-1" >}}
```

Overall, it seemed too complicated or I am getting lazy. I could get it migrated to Github Actions and have automated rendering at every git push. Maybe spend some time researching better organization of files and best practices.

## Migrating

Instead I searched in Goggle: "Migrating from Hugo to"

![](gsearch.png)

What options popped up?

1. [Gatsby](https://www.gatsbyjs.com/) - Javascript based
2. [11ty](https://www.11ty.dev/) - Javascript based but it could use Markdown templates with some plugins
3. [Quarto](https://quarto.org/) - Documents as plain text markdown or Jupyter notebooks.

First two felt like going back [Jekyll](https://jekyllrb.com/) but worse because it will use Javascript instead of Ruby. But third option sounded promising.

## Comparison

|| Hugo | Quarto
---------|----------|---------
 Markdown templates | &check; | &check;
 Front Matter | TOML[^1] | YAML 
 Shortcodes | &check; |  &check;
 Github integration | &check; |  &check;
 CLI | &check; | &check;
|Config | TOML| YAML
| New site generator | &check; | &check;[^2]
| New page generator | &check; | &cross;
| tags | &check;[^3] | &cross;
| categories | &check; | &check;
| Server with live reload | &check; |  &check;

[^1]: Originally when I started but today I guess it supports TOML, YAML and JSON
[^2]: Hugo got site generator while Quarto have projects and one of the project types is website or blog
[^3]: I was always confused why I need to define both in Hugo

On very basic level both tools looked similar, same type of input, simple project structure and static site as an output. 

# New Quarto workflow.

Started with generated empty blog:
```
quarto create-project melastmohican.github.io --type website:blog
```

In Hugo, my .md files were in `content/posts` as individual files, rather than  in folders. For my Quarto site, I wanted them in a folder called `posts` and I wanted each post to have it’s only folder with content in a file called index.qmd nested within. I wrote a quick python script to help me do this.

``` python
#!/usr/bin/env python3
import os
import glob
from pathlib import Path
import shutil

# Get the list of all files and directories
path = "<hugo site>/content/posts/"
files = glob.glob(path + "*.md")

for i in range(len(files)):
    src = files[i]
    foldername = Path(src).stem
    dir = "posts/"+foldername
    Path(dir).mkdir(parents=True, exist_ok=True)
    dst = dir+"/index.qmd"
    print("src: " + src + " dst: " + dst)
    shutil.copy2(src, dst)
```

I have to do some cleanup after that but project folder looks almost like in Hugo:

``` bash
.
├── README.md
├── _quarto.yml
├── _site
│   ├── about.html
│   ├── index.html
│   ├── listings.json
│   ├── posts
│   ├── profile.jpg
│   ├── robots.txt
│   ├── search.json
│   ├── site_libs
│   ├── sitemap.xml
│   └── styles.css
├── about.qmd
├── index.qmd
├── posts
│   ├── _metadata.yml
│   ├── migrating-from-hugo-to-quarto
│   │   ├── gsearch.png
│   │   └── index.md
├── profile.jpg
└── styles.css
```

1. If I want new post:
``` bash
mkdir posts/migrating-from-hugo-to-quarto
touch posts/migrating-from-hugo-to-quarto/index.qmd
```

2. Then I need to add Front Matter:
``` yaml
---
title: "Migrating my hugo blog to quarto"
subtitle: | 
  I have moved this blog from hugo over to quarto, and taken notes.
date: "2022-11-20"
categories: [quarto, blogging, hugo]
image: "preview.jpg"
---
```

3. Next, I can try it:
``` bash
quarto preview
```

4. Last step would be:
``` bash
quarto publish gh-pages
? Update site at https://melastmohican.github.io? (Y/n) › Yes
From https://github.com/melastmohican/melastmohican.github.io
 * branch            gh-pages   -> FETCH_HEAD
Rendering for publish:
[ 2/12] posts/migrating-from-hugo-to-quarto/index.md

branch 'gh-pages' set up to track 'origin/gh-pages'.
HEAD is now at a871c16 rebuilding site Mon Nov 14 16:32:05 PST 2022
Preparing worktree (resetting branch 'gh-pages'; was at a871c16)
[gh-pages 6750867] Built site for gh-pages
 39 files changed, 11245 insertions(+), 2402 deletions(-)
 create mode 100644 posts/migrating-from-hugo-to-quarto/gsearch.png
 create mode 100644 posts/migrating-from-hugo-to-quarto/index.html
 create morigin https://github.com/melastmohican/melastmohican.github.io.git (fetch)
origin  https://github.com/melastmohican/melastmohican.github.io.git (push)
To https://github.com/melastmohican/melastmohican.github.io.git
   a871c16..6750867  HEAD -> gh-pages

NOTE: GitHub Pages sites use caching so you might need to click the refresh
button within your web browser to see changes after deployment.

[✓] Deploying gh-pages branch to website (this may take a few minutes)
[✓] Published to https://melastmohican.github.io
```