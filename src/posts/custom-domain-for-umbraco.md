---
title: Using a custom domain for accessing umbraco
metaDesc: Changing how your content editors access Umbraco.
date: '2015-03-16'
tags: 
- umbraco
---

By default, content editors will be editing their content by appending umbraco to their domain. Sometimes you might not want your content editor to use that method and would like them to use an alternative URL. This could be for a few reasons...

- You don't want Joe public trying to access the Umbraco back-office on your site. This is more security for obscurity and therefore isn't necessarily a perfect solution, but by changing the link to Umbraco, it does partially lock it down. Note, if you set a hostname on your server to the URL you want to use and then point your hosts file to the server it is possible to prevent /umbraco from being publically accessible. i.e. it is only local to the server.
- You want to create a different URL for your content editor to use when editing content. Perhaps it might be nicer for them to use or they only want the Umbraco side of things to be under SSL.
- You have multiple sites within one Umbraco instance. This actually makes it a lot easier to group sites into one entity, otherwise each site will have their own Umbraco URL. One URL to remember, one URL to use.

Below is an example of how to do this. We match the URL for Umbraco and then perform a redirect to our not found page if it doesn't match our pattern. Note this part: **(?!Surface)** - this is important as it means that AJAX calls using Surface controllers still work.

Therefore, when editing the Umbraco site and assuming will only have an SSL certificate for Umbraco, a content editor will use:

> https://admin.example.com/umbraco/  

Whereas, the site will be visible at:

> http://www.example.com/ 

If you have done something similar or have alternatives that you use, then please feel free to leave a comment! 

```xml
<!-- Restrict access to Umbraco -->
<rule name="Restrict access" stopProcessing="true">
    <match url="umbraco(?!/Surface/)" />
    <conditions logicalGrouping="MatchAny" trackAllCaptures="false">
        <add input="{HTTP_HOST}" matchType="Pattern" pattern="admin.example.com" ignoreCase="true" negate="true" />
    </conditions>
    <action type="Redirect" url="/not-found" appendQueryString="false" />
</rule>
```
