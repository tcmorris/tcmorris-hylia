---
title: Upgrading Umbraco
tagline: A rundown of some of the reasons for upgrading Umbraco and the challenges faced.
tags: umbraco
---

During a recent project, I was tasked with upgrading a site from v4 of Umbraco to the latest and greatest. There are a few reasons for this and it doesn't come without it's challenges, but I think the end result is definitely worth it.

<sub>
Note: For reference, I am talking mainly around upgrading an existing implementation of Umbraco and not migrating to another instance. Also, specifically around major releases, since patch updates rarely cause any issues and where possible you should always keep up to date if you can.
</sub>

### Why Upgrade?

Well, who wouldn't want to make use of the updated UI and far improved underlying architecture of Umbraco. There are many things which the HQ team (and others within the community) have been working on and a lot can change in a relatively short time in the tech world.

- Complete redesign
- Works in mobile formats
- Better editing experience
- New service layer
- Better performance
- Greater support for latest technology
- Packages!
- etc.

On the other hand, there may be problems around how the site used to be built (think WebForms/XSLT) which render it difficult to upgrade or simply too time consuming to get back on to the upgrade path and work with the **best version** of Umbraco. These are choices which are largely down to the client and will require some conversations around. Obviously, we'd all like to work to the latest version and be on a common playground, but sometimes other things hinder us from taking these steps.

The good news is that with some planning and a little bit of perserverance and knowledge it isn't actually as daunting as you think it might be. I've heard many a time that the concept of upgrading a site that was built 3-5 years ago was better off with a complete rewrite and rework of the content, including fresh designs. Whilst this might be the path you go down more often than not, sometimes it's not exactly feasible or best placed to migrate a bunch of content and then factor in all of the changes around that. For larger sites, this certainly becomes the case.

### What's the process?

Since 7.3, largely an Umbraco upgrade can be done via NuGet without too much extra work required. This is all dealt with by the migration scripts that were added and are included with each minor release from now on (even dating back to [FourOneZero](https://github.com/umbraco/Umbraco-CMS/tree/dev-v7/src/Umbraco.Core/Persistence/Migrations/Upgrades/TargetVersionFourOneZero)). When you update all the files to the latest version, Umbraco will perform a check against the database and if it finds that it needs to update then you will be prompted to upgrade the database via the installer. Click the button and wait until you see the new Umbraco UI.

But things might not go as smooth as that, sometimes the installer might fallover or it will report that there are incompatible data types. What do you do then? When it comes to the data types, I found that you could map these to the newer counterparts or simply take a note and update those once you have performed the base upgrade. Take a note of what was there before and what you want to have afterwards. As for when it fails, find the error and look for support on the forums, or place an issue on the tracker. Everyone that does this, will help Umbraco to help others and make the process better for all.

At this point I'm referring mostly towards the concept of getting Umbraco working in terms of the back-office and having a working CMS. We still need to take a look at our code that actually runs the website. Above I mentioned that you might be using old technology or you might be using packages / data types that are no longer compatible. The prep work around figuring out this stuff before is important and will provide a decent comparison between where you are and where you want to be. I found that exporting the doc types, templates and macros from the database was a particularly useful exercise for this. It's quite possible that there aren't many custom implementations and that actually once you've done the database side of things, Umbraco works without any major adjustments.

Some of the other things you might want to look into:

- update the icons
- update data types (if required)
- figure out best use of packages
- assess the use of doc types (it's possible to change them and sort into folders)
- assess the use of macros (could it just be a Razor partial?)
- check over the content

Switching out WebForms/XSLT for MVC/Razor can be a time consuming task, but in the end it turns out to be an opportunity to amend some of the methods from before and update to new techniques. It's also a rather nice way of realising that XSLT was rather nasty, and in getting rid of it all you can brief a sigh of relief that another Umbraco site has moved on from the not so distant past. Figuratively speaking of course. Unfortunately, you won't be able to switch out too much when it comes to content structure, but you may be able to upgrade some of those old packages to their equivalent v7 counterparts. For example, Embedded Content now becomes Nested Content and everyone is happy.

### Conclusion

So, you've got an old site and are thinking about taking the leap. Assess your options, figure out what is important and how big the scale of the job might be. Take backups and compare. Embrace the changes and enjoy the new world of Umbraco. It's quite a good CMS you know.
