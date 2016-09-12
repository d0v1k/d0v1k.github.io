---
published: true
layout: post
comments: true
title:  "Adding Disqus to Jekyll"
date:   2016-09-12
categories:
  - Howtos
  - blog
---

Adding Disqus to your Jekyll Blog is quiet easy, I will show you how you can do it on you jekyll website.

Open _data/theme.yml and add the following code. Remember to change username to your own Disqus shortname.

# Disqus post comments
disqus_shortname: username

Create a file called disqus.html in Jekyll’s _includes folder and add your Disqus Universal Embed Code in between a {% if page.comments %} and a {% endif %} liquid tag.

{% raw %}{% if page.comments != false %}{% endraw %}
```
<div id="disqus_thread"></div>
<script>
    /**
     *  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
     *  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables
     */
    var disqus_config = function () {
        this.page.url = 'http://d0v1k.github.io{{ page.url }}';  // Replace PAGE_URL with your page's canonical URL variable
        this.page.identifier = '{{ page.id }}'; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
    };
    (function() {  // DON'T EDIT BELOW THIS LINE
        var d = document, s = d.createElement('script');
        
        s.src = '//d0v1k.disqus.com/embed.js';
        
        s.setAttribute('data-timestamp', +new Date());
        (d.head || d.body).appendChild(s);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
{% endif %}
```

The included if statement allows you to disable Disqus comments on any blog post. You simply add comments: false in that posts front-matter.

Finally, open your post.html file and add the following liquid include tag just after the end </div> tag. This will load Disqus comments right underneath your blog posts.

```
{% if site.data.theme.disqus_shortname %}
{% include disqus.html %}
{% endif %}
```

Refresh your page click on one of your post you should see your Disqus comments
