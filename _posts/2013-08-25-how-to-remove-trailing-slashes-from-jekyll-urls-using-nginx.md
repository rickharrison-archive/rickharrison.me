---
layout: post
title: How to remove trailing slashes from Jekyll URLs using Nginx
---

Recently, I started using [Jekyll](http://jekyllrb.com/) to generate my blog and site. I wanted to have clean URLs without a .html extension or a trailng slash at the end. Unfortunately, jekyll makes it difficult to accomplish both of these objectives simultaneously.

The easiest way to do this is to have jekyll think you are serving up files with the .html extension and then remove it using Nginx. To do this, you need to have a permalink setting in your `_config.yml` to be similar to the following:

```yaml
permalink: /:title.html
```

Prior to telling Nginx to serve up the files without the .html extension we need to have Jekyll generate links without .html to avoid any unnecessary 301 re-directs. All of the links will need to have `| remove '.html'`.

{% raw %}
```html
<a href="{{ post.url | remove: '.html' }}">
```
{% endraw %}

That is all of the configuration Jekyll will need. Now everything is on Nginx to serve it correctly. First, set up a simple server block for the domain:

```nginx
server {
    listen 80;

    root /var/www/yourdomain.com;
    index index.html index.htm;
    server_name yourdomain.com;
    access_log /var/log/nginx/yourdomain.com.log;

    location / {
        try_files $uri.html $uri $uri/ /index.html;
    }
}
```

In order to remove all of the trailing slashes from the URL, we will want to make sure that the user is not trying to access a real directory. To do that we check for an index.html in the current path (I.E. if the current path is /directory/ we will check for /directory/index.html). If there is no index, a 301 is performed to take the last slash off. In Nginx, `!-f` means "file does not exist".

```nginx
if (!-f "${request_filename}index.html") {
    rewrite ^/(.*)/$ /$1 permanent;
}
```
The next step is removing any index.html from the URL. That way, a URL such as yourdomain.com/index.html can be redirected to yourdomain.com

```nginx
if ($request_uri ~* "/index.html") {
    rewrite (?i)^(.*)index\.html$ $1 permanent;
}
```

Finally, a 301 redirect will be set up to remove any .html extension from the URL (I.E. from /blog.html to /blog). Once again, this is a simple if statement with a rewrite statement:

```nginx
if ($request_uri ~* ".html") {
    rewrite (?i)^(.*)/(.*)\.html $1/$2 permanent;
}
```

That's it! The only key thing to remember is to remove all of the .html extensions from links when creating jekyll templates/pages using the `| remove '.html'` filter. Another trick I like to do is remove the www subdomain from all urls. This can be done with an extra server block that looks for `www.yourdomain.com` and then redirects it. Combining the two server blocks results in a small config ready to go for clean URLs in jekyll.

```nginx
server {
    listen 80;
    server_name www.yourdomain.com;
    return 301 $scheme://yourdomain.com$request_uri;
}

server {
    listen 80;

    root /var/www/yourdomain.com;
    index index.html index.htm;
    server_name yourdomain.com;
    access_log /var/log/nginx/yourdomain.com.log;

    if (!-f "${request_filename}index.html") {
        rewrite ^/(.*)/$ /$1 permanent;
    }

    if ($request_uri ~* "/index.html") {
        rewrite (?i)^(.*)index\.html$ $1 permanent;
    }

    if ($request_uri ~* ".html") {
        rewrite (?i)^(.*)/(.*)\.html $1/$2 permanent;
    }

    location / {
        try_files $uri.html $uri $uri/ /index.html;
    }
}
```
<p class="close-link"><a href="https://gist.github.com/rickharrison/6410194">View as a Gist</a></p>
