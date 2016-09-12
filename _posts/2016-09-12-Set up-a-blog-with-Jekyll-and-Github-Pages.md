---
published: false
---
---
layout: post
title: Set up a blog with Jekyll and Github Pages
categories:
- blog
---

Update Ubuntu

<code>sudo apt update</code>

Install Ruby package

<code>sudo apt install ruby-full</code>

Install Jekyll package with gem

<code>sudo gem install jekyll bundler</code>

Verify version of jekyll to ensure whether it is installed or not

<code>sudo jekyll -v</code>

Change to home directory

<code>cd /home/yourusername</code>

Create a new website

<code>sudo jekyll new yournewsite</code>

The above command will create a new web directory under /home/yourusername

<code>cd /home/yourusername/jekyllsite
sudo bundle install</code>

*If css not loading

Edit _config.yml and change as below

<code>url: "http://youripaddress:4000"</code>

Time to start jekyll application, replace below mentioned ip address with your ip.

<code>sudo bundle exec jekyll server --host youripaddress</code>





Enter text in [Markdown](http://daringfireball.net/projects/markdown/). Use the toolbar above, or click the **?** button for formatting help.
