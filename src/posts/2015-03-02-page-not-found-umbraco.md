---
title: Page Not Found in Umbraco
tagline: A quick and easy example of how to implement a page not found content finder in Umbraco.
tags: umbraco
---

Content finders in Umbraco are really useful as they can intercept the routing and give you access to the context. There are a few uses of this and one on those is to find your page not found page that you have created especially in Umbraco. This differs from the more traditional way of setting a page id within your umbracoSettings.config file and is useful for giving your client complete control over their page not found page. So, in the example below, there would be a corresponding document type in Umbraco called **NotFoundPage**.
