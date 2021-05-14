---
layout: post
title:  "Setup your own blog post using Jekyll and github"
author: Dhanesh Padmanabhan
date:   2021-05-14
tags: jekyll getting-started
---
This is a hello world post, and it is apt that it has to be about Jekyll that was used to set up this website.

Jekyll is a static page generator with a nice and simple format. The content goes in markdown files. It can be very easily hosted on github with very minimal setup. It is a perfect way to just host some blogs or software documentation. 

There are lots of useful information online on how to get started with a basic jekyll blog post with a `minima` theme like [this one][jekyll-getstarted]. I would also recommend  reading the [jekyll documentation][jekyll-doc] to know about the basic concepts of Jekyll like liquid templating. 

I started with `minima` theme but was looking for a nicer theme, and switched to `jekyll-theme-minimal`. The `jekyll-theme-minimal` does not provide default header with "About" links or a list of posts like `minima` does. I just manually linked the rendered `aboutme.html` in `index.html`. Note: I use an `index.html` instead of `index.markdown` to make pagination work. For getting the list of posts, I borrowed some liquid code from `minima` home layout and created a custom home layout under `_layouts` folder. After doing this, I just changed the layout in `index.html` to `home` . I also created my own logo and put in `./assets/img/logo.png`. So jekyll is quite nice indeed. In about a day's time, I have a nicely customized blog post, and I can keep adding more content quite easily in markdown.

You can just fork this repo, edit the `_config.yml`, `aboutme.markdown` and `index.html` and start adding your posts under `_posts` folder in markdown syntax. You should install ruby, gem and jekyll on your machine and do a dry run on your machine using the following command:

```bash
bundle exec jekyll serve
```
This will serve the site on http://localhost:4000/blog. Once you are happy with the edits, you can publish to your github. 

If you have any suggestions, please e-mail me or submit a PR to this code. Thanks for reading my first blog.

[jekyll-getstarted]: https://programminghistorian.org/en/lessons/building-static-sites-with-jekyll-github-pages
[jekyll-doc]: https://jekyllrb.com/docs/




