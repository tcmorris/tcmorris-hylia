---
title: Enabling features in production
metaDesc: An approach to turn website features on/off in production
date: '2019-12-18'
tags: 
- umbraco
- dotnet
---

**Note: this article was written for 24days in Umbraco and the original can be found here: [https://24days.in/umbraco-cms/2019/features-in-production/](https://24days.in/umbraco-cms/2019/features-in-production/)**

When deploying features or new changes to our websites, we might not want them to be available to all users at once. It would be nice if we could slowly introduce new features to our users. We might even want to completely disable them and have them there in secret. In this article, I'm going to present a few different ways in which you can release features into production, even when they might not be fully complete.

## Background

Before we look into the ways in which we can get our features into production, it's useful to think about how we might have typically developed new changes in our website before a release.

One of the most common ways is to use feature branches in a [gitflow scenario](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow), where you would build your new feature, deploy to a test environment and then try and merge those changes with anything else that has changed in the meantime. You'll probably bundle a number of changes together and then release those to production.

Some of the issues at this stage might be that you've got a tricky merge, you've tested something in isolation and when you bring it all together other issues might arise. You also haven't really trialled those changes with actual users, so those changes may not be desired or there could be problems with how it all works.

## Benefits and options

Therefore, it's probably a good idea to highlight some of the benefits as to why you might want to integrate your new changes sooner rather than later.

- your features get into production quicker
- you can get feedback earlier
- you can test things in reduced quantities
- you should end up with a better product
- your client will be happy

You might have seen reasons like this elsewhere, and that's because they actually run true for agile development and the scrum process, which in turn is a good fit for [continuous delivery](https://continuousdelivery.com/). A notion that your app is always in a deliverable state, since any new changes have been safely integrated somehow.

On to the ways in which we can enable our changes...

### App settings

One of the easiest ways is to define in your web.config, application settings somewhere, or provide settings in your Umbraco solution to features on and off. You'll need to check this setting within your code before including the new changes for your users. 

This can be done with a feature helper, which you can query the app setting and then return a true or false as to whether the new changes should be included or not. 

```csharp
public static class FeatureHelper
{
    public static bool IsFeatureEnabled(string featureFlag)
    {
        // check if we have enabled functionality via web.config
        bool.TryParse(ConfigurationManager.AppSettings.Get(featureFlag), out var setting);
        return setting;
    }
}
```

If you'd like to turn off entire controllers or action methods, one of the ways in which you can do that is via the use of [attributes](https://docs.microsoft.com/en-us/aspnet/mvc/overview/older-versions-1/controllers-and-routing/understanding-action-filters-cs). These will decorate your actions and apply the filter before any of the other code is run. In the attribute logic, it will call our feature helper to determine whether or not a feature should be applied. 

```csharp
public class FeatureAttribute : ActionFilterAttribute
{
    public string ConfigVariable { get; set; }        
    
    public FeatureAttribute(string configVariable)
    {
        ConfigVariable = configVariable;        
    }

    public override void OnActionExecuting(ActionExecutingContext filterContext)
    {
        // if the feature is not enabled, then redirect to 404
        if (!FeatureHelper.IsFeatureEnabled(ConfigVariable))
        {
            filterContext.HttpContext.Response.Redirect("/404/");
        }
        else
        {
            base.OnActionExecuting(filterContext);
        }
    }
}
```

This allows for features to be controlled relatively easy in terms of when it is available, but it's an all or nothing solution which might not be desired when you want to tentatively introduce customers to your new feature.

### Session

We can apply changes based on a user's session data, which could be enabled via a campaign to help with testing. Ask for people to click a link and become part of the session. You can then trigger an update that will kill off any session data if there are issues. The code above that we used for checking app setting can be amended to apply with session data.

```csharp
public static class FeatureHelper
{
    public static bool IsFeatureEnabled(string sessionFlag)
    {
        // check if we have a valid session to enable functionality
        var sessionVar = HttpContext.Current?.Session[sessionFlag];
        if (sessionVar == null)
        {
            return false;
        }

        return bool.Parse(sessionVar.ToString());
    }
}
```

This is great for ad hoc testing, as everything can return to normal after the session. This is ideal if you don't have an account system set up, but it is helpful if you can target or segment your users if you have a hypothesis you want to test. 

A/B testing would be a good scenario here. One set of users could get the feature enabled and the other set could continue as normal to see if anything is improved if your feature relates to performance or conversions. It's important that you can define what improvement looks like or can at least get some feedback from your users.

### User preference

If you do have an account section, you could provide the user the choice as to whether they want to opt in to a newly released feature that you might want to test further before enabling for all. You would need to check against the current logged in user as to whether or not they would like to see those changes.

In Umbraco, you could define this as a member property or [membership group](https://our.umbraco.com/Documentation/Getting-Started/Data/Members/#creating-member-groups) so you can see within the backoffice which users have enabled your feature and segment your users that way. If your changes are more granular, then using properties is probably preferred.

_Just to note, for members in Umbraco there is the concept of role based access, which is tied with membership groups, so could be an option if you wanted to hook in that way instead. This uses .NET authorisation under the hood._

![Umbraco member properties](/images/umbraco-membership-features.png "Image with list of features that can be enabled for an Umbraco member")

This puts the control on to your users, and they are more likely to be actively engaged with a feature and also understand that there may be issues. If they're not happy with how it works, then they can easily opt back out and carry on as normal. Within your website, you can provide an option for feedback based on whether a new feature has been turned on. They can submit reports and you can gain some quality insight as to how your users are actually using your changes.

### Deployment slots

If you're doing your deployments via Azure App Services, then another option at your dispense, is to use [deployment slots](https://docs.microsoft.com/en-us/azure/app-service/deploy-staging-slots#route-traffic). The main use case for this is to run a staging slot alongside your live slot, and to swap them when doing a deployment to ensure that you've tested your changes, warmed up your site beforehand and traffic is switched over without downtime. 

There is another benefit to it though, in that it can allow you to drive traffic to either slot based on a percentage. That way you can test with 20% of users before rolling out to the other 80% and this is managed through deployment. You can do similar with traffic manager or other load balancing rules. 

![Azure deployment slots](/images/deploymentslots-routetraffic.png "Image showing the options for routing in Azure deployment slots") 

With this option, it's even more important that you can provide suitable metrics and logs to determine how your new changes are performing. Without reasonable guidance, you're in the dark as to whether or not it's a good idea to roll out.

### Features as a service

With the rise of microservices, why not make use of a service that can handle our features? One of those services is [LaunchDarkly](https://launchdarkly.com/), which provides a dashboard for your features and has a number of libraries (inc. JS and .NET) that you can integrate with. As a solution, it'll allow you to get a full feature management tool pretty quickly, and one that should provide plenty of options albeit at a price. Useful to consider, and there may well be others out there. The concepts should still be the same as we've outlined in this article. 

**[Update]**

Since originally writing this article, another option which I've found is the [.NET Feature Management](https://github.com/microsoft/FeatureManagement-Dotnet) library. This is built on top of the .NET configuration system and provides a nice set of APIs for using in your applications. It looks like a great option. Here's a [quick start guide](https://docs.microsoft.com/en-us/azure/azure-app-configuration/quickstart-feature-flag-dotnet) to set up with Azure.

## Conclusion

Testing features in production is a big part of continuous improvement and ties in well with the continuous development way of doing things. You can deliver changes quicker and have greater confidence in when a feature is enabled for all. Ideally, you can ramp up usage in a controlled manner.

You can gain feedback on your changes from real users and scenarios, which may in turn provide much greater insight as to how something works. The end goal being that your changes have been well integrated and are of better quality overall.

