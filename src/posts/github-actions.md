---
title: GitHub Actions
metaDesc: How to configure GitHub Actions to build, package and deploy your project.
date: '2020-08-19'
tags: 
- dotnet
- github
---

There are quite a few build and deploy options available to developers these days. Previously, I have wrote about using a combination of [Team City and Octopus Deploy](http://mozzy.dev/posts/continuous-delivery-for-dotnet-revisited/). These are still good tools, but will likely require a bit of setup and probably require a VM. 

A more recent trend is to have your actions linked to your repository, where you can have it all self contained and in one place. There are pros and cons to both, but I'm gonna show you how you might do that with GitHub Actions.

## dotnet

I work mostly with .NET, so lets take a look at the workflow for that. With .NET Core you can now use Linux (and macOS) as your build target. Here we're using `ubuntu-latest`. 

We're also setting the relevant `DOTNET_VERSION` and running our commands, `dotnet build` and `dotnet publish`. Problems with the build will be visible within GitHub and you can follow along with the progress.

Finally, we're pushing our published version of the app to Azure Web Apps using a publish profile. You could also choose to generate a NuGet package and push to a feed for distribution. 

<script src="https://gist.github.com/tcmorris/dada56a630316670a064882a2753f04f.js"></script>

Note, you can also build [.NET Framework apps](https://github.com/Azure-Samples/dotnet-sample/blob/master/.github/workflows/aspnet.yml), that will look a little different but the concept is the same. You'll target `windows-latest` as that is a prerequisite for .NET Framework, and the commands will be `msbuild`.

## node.js

If you've got a FE repo, then you'll likely want to use npm to compile some assets and package them up. Here's how you can do that. 

We've set up our `NODE_VERSION` and then run the commands, `npm install` and `npm build`. At that point, we might want to package up the assets or deploy as an application. 

<script src="https://gist.github.com/tcmorris/a779e0c62f9db194b7a15bdef0ec696e.js"></script>

## other

There are lots of other options available to you, without going into all of them here, I encourage you to take a look through the [docs](https://docs.github.com/en/actions/language-and-framework-guides) and see what you might want for your needs. 

If you're interested in what's installed on the runners, then take a look at [this repo](https://github.com/actions/virtual-environments) which contains the full list of software and spec for each.

The approach here can also be used for similar tools such as [Bitbucket Pipelines](https://bitbucket.org/product/features/pipelines) or [Azure Pipelines](https://azure.microsoft.com/en-us/services/devops/pipelines/). Choose what fits the bill for you.

Have fun!

