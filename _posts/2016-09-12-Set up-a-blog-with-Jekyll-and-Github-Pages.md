---
published: true
layout: post
categories:
  - blog
---
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





Enter text in [Markdown](http://daringfireball.net/projects/markdown/). Use the toolbar above, or click the **?** button for formatting help.
