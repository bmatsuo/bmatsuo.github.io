---
layout: post
title:  "Integrating bootstrap with jekyll"
date:   2013-06-23 16:51:13
categories: jekyll update
---

Download bootstrap, unzip it and distribute the files within the bootstrap hierarchy

{% highlight bash %}
wget http://twitter.github.io/bootstrap/assets/bootstrap.zip
unzip bootstrap.zip
rm bootstrap.zip
{% endhighlight %}

Edit `_layouts/default.html` and include bootstrap files

{% highlight html %}
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
        <title>{{ page.title }}</title>
        <meta name="viewport" content="width=device-width">

        <!-- bootstrap -->
        <link rel="stylesheet" href="/bootstrap/css/bootstrap.css">

        <!-- syntax highlighting CSS -->
        <link rel="stylesheet" href="/css/syntax.css">

        <!-- Custom CSS -->
        <link rel="stylesheet" href="/css/main.css">

    </head>
    <body>

        <div class="container">
          <div class="site">
            <div class="header">
              <h1 class="title"><a href="/">{{ site.name }}</a></h1>
              <a class="extra" href="/">home</a>
          </div>

                {% assign content = '{{ content }}' %}{{ content }}

            <div class="footer">
              <div class="contact">
                <p>
                  Bryan Matsuo<br />
                </p>
              </div>
              <div class="contact">
                <p>
                  <a href="http://github.com/bmatsuo/">github.com/bmatsuo</a><br />
                </p>
              </div>
            </div>
          </div>
        </div> <!-- /container -->

        <script src="http://code.jquery.com/jquery.js"></script>
        <script src="/bootstrap/js/bootstrap.js"></script>
    </body>
</html>
{% endhighlight %}
