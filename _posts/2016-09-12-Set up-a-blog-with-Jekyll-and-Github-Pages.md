---
published: true
layout: post
comments: true
title:  "Set up a blog with Jekyll and Github Pages.md"
date:   2016-09-12 14:35:57 +0200
categories: Howtos
  - blog
---


categories: jekyll disqus

Update Ubuntu

	sudo apt update

Install Ruby package

    sudo apt install ruby-full

Install Jekyll package with gem

	sudo gem install jekyll bundler

Verify version of jekyll to ensure whether it is installed or not

	sudo jekyll -v

Change to home directory

	cd /home/yourusername

Create a new website

	sudo jekyll new yournewsite

The above command will create a new web directory under /home/yourusername

	cd /home/yourusername/jekyllsite
	sudo bundle install

*If css not loading

Edit _config.yml and change as below

	url: "http://youripaddress:4000"

Time to start jekyll application, replace below mentioned ip address with your ip.

	sudo bundle exec jekyll server --host youripaddress

## How to host on github

Create a New Repository

    Go to your https://github.com and create a new repository named USERNAME.github.io

Change to your jekyllsite directory

    cd /home/yourusername/jekyllsite
    git remote set-url origin https://github.com/USERNAME/USERNAME.github.io.git
    git push origin master

Publish

After you've added posts or made changes to your theme or other files, simply commit them to your git repo and push the commits up to GitHub.

    git add .
    git commit -m "New Config"
    git push origin master 



