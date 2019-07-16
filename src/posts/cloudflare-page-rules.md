---
title: Cloudflare page rules
metaDesc: A quick overview of Cloudflare page rules and setting up a 301 redirect.
date: '2019-07-16'
tags: 
- cloudflare
---

As you might have read, I recently moved my site over to a new domain. [mozzy.dev](https://mozzy.dev)

I needed a way to redirect requests from my old domain to the new one, but I didn't really want to go creating redirect rules at the server level or whatever (think IIS, Apache). In fact, [tcmorris.net](https://tcmorris.net) is simply a Github Pages site. It needed to be a 301 redirect and not just a [meta refresh](https://developer.mozilla.org/en-US/docs/Web/HTTP/Redirections#HTML_redirections) tag, which simply reloads the browser.

Way back, I used Cloudflare to provide free SSL on my site, and the DNS etc is still handled there. So, I had a look around and came across Page Rules. These are neat little rules that you can apply to your app such as 'Always use HTTPS' or 'Auto Minify'. A lot of this is provided by Cloudflare to use across your site, but this allows you to be specific with individual pages if you wish.

The one that I was interested in was 'Forwarding URL', which allows you to forward requests and trigger a redirect of some sort (301/302). You can use wildcards in the matching rule to cover all pages for example, and then set a new destination along with the matching wildcard info. Example below for how I managed to get a 301 redirect working to my new domain. 

![Cloudflare Page Rule example](/images/cloudflare-page-rules.jpg "Setting up 301 redirect in Cloudflare Page Rules.")

Rather neat I'm sure you'll agree. 

For more info, have a look at their docs: [Cloudflare examples](https://support.cloudflare.com/hc/en-us/articles/218411427)
