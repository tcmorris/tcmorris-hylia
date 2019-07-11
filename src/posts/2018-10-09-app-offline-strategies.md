---
title: App offline strategies for your website
tagline: What options do we have for putting a website under maintenance?
tags: umbraco dotnet
---

It's undesirable, but sometimes you'll be in a scenario where you may need to take your sife offline for maintenance. This could be an upgrade to your CMS of choice (e.g. [Umbraco](https://umbraco.com/)), processing of changes behind the scenes or simply restricting users at high traffic.

### app_offline.htm

The simplest way to do this is to place an [app_offline.htm](https://docs.microsoft.com/en-gb/iis/publish/deploying-application-packages/taking-an-application-offline-before-publishing) file in the root directory of the website. This will instruct .NET to route all traffic to that file and show this to the user. It'll need to be simple html and you'll need to inline any CSS that you want applied.

This is great if you want to restrict traffic altogether, but if you want to do stuff in the background then you are a bit stuck.

### IIS Rewrite

So, one way in which you can get around that is by using a rewrite rule which negates a certain IP. This will allow whoever visits your site via the IP added, to browse the website without obstruction. Anyone else, will get served the offline file. Below is an example transform that you can use to achieve that.

``` xml
<rule name="App-offline" stopProcessing="true" enabled="true" xdt:Transform="InsertIfMissing" xdt:Locator="Match(name)">
    <match url="^(.*)$" ignoreCase="false"/>
    <conditions logicalGrouping="MatchAll">
        <add input="{URL}" pattern="/offline.htm$" ignoreCase="false" negate="true"/>
        <add input="{REMOTE_ADDR}" pattern="^127.0.0.1$" ignoreCase="false" negate="true"/>            
    </conditions>
    <action type="Redirect" url="/offline.htm" redirectType="Found"/>
</rule>
```

### Maintenance manager

Perhaps you don't have a specific IP you want to allow and are interested in being able to turn this off and on. Well, the good news is if you are using Umbraco, then Kevin Jump has created a package for that. It's called maintenance mode for Umbraco. You can login to Umbraco and then you'll be given the option to apply the restriction to everyone, as well enabling access for back office users if you wish. 

You can download that here: [https://our.umbraco.com/packages/backoffice-extensions/maintenance-manager/](https://our.umbraco.com/packages/backoffice-extensions/maintenance-manager/)

### Limited downtime?

There's a few ways to achieve this. One of the most common being [blue-green deployments](https://octopus.com/docs/deployment-patterns/blue-green-deployments) where you alternate between active and inactive versions of the website, resulting in one being live and the other being updated and switched to. Usually this will require some kind of load balancer although with deployment slots in Azure you can achieve a similar thing.

You could use tooling such as [Octopus Deploy](https://octopus.com/) to set up a variety of deployment strategies, with the ability to run custom scripts and actions throughout the deployment. You can also configure a deployment to run at scheduled times (e.g. 2am where there is expected to be little traffic). 
