---
title: Page Not Found in Umbraco
metaDesc: A quick and easy example of how to implement a page not found content finder in Umbraco.
date: '2015-03-02'
tags: 
- umbraco
---

Content finders in Umbraco are really useful as they can intercept the routing and give you access to the context. There are a few uses of this and one on those is to find your page not found page that you have created especially in Umbraco. This differs from the more traditional way of setting a page id within your umbracoSettings.config file and is useful for giving your client complete control over their page not found page. So, in the example below, there would be a corresponding document type in Umbraco called **NotFoundPage**.

```csharp
using System.Linq;
using Umbraco.Core;
using Umbraco.Core.Models;
using Umbraco.Web;
using Umbraco.Web.Routing;

public class PageNotFoundContentFinder : IContentFinder
{
    public bool TryFindContent(PublishedContentRequest request)
    {
        // have we got any content?
        if (request.PublishedContent == null)
        {
            // Get the root node for domain
            var home = request.RoutingContext.UmbracoContext.ContentCache.GetByRoute(request.Domain.RootNodeId + "/");

            // Try and find the 404 node
            var notFoundNode = home.Children.Where(x => x.DocumentTypeAlias == "NotFoundPage").FirstOrDefault();
            if (notFoundNode != null)
            {
                // Set Response Status to be HTTP 404
                request.SetResponseStatus(404, "404 Page Not Found");

                // Set the node to be the not found node
                request.PublishedContent = notFoundNode;
            }
        }

        // hopefully we will have content at this point
        return request.PublishedContent != null;
    }
}
```
