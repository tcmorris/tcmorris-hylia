---
title: Continuous delivery for .NET (revisited)
tagline: An update on using TeamCity and Octopus Deploy to package up and deploy applications.
tags: 
- teamcity 
- octopusdeploy 
- dotnet
---

Last year I wrote about deployments and the idea of [continuous delivery for .NET](/blog/continuous-delivery-for-dotnet/). During that article I spoke about how to set up and configure TeamCity and Octopus Deploy as tools for deployment as well as adding some notes around process. 

Well, things change and constant improvements are made, so this is a look back on that article and an update as to how the process has been modified. The good news is that the tools chosen in the first article have become staples in the deployment process and what we are talking about here is a refinement. To reiterate the purpose... 

> TeamCity is the build tool, Octopus Deploy is the deployment tool.

## TeamCity

As mentioned before we are using TeamCity to build the solution, perform tests and create an artifact for deployment. We also use a template so this can be shared across projects.

1. Configure version ([GitVersion](https://github.com/GitTools/GitVersion))
2. Fetch packages, build solution and run OctoPack
3. Front end tasks (e.g. npm install / grunt / gulp)
4. Perform tests and check code coverage
5. Publish package to Octopus Deploy NuGet server

You may want to split this out into 2 or 3 configurations depending on requirements. Reasons for doing this would be if you wanted to run your unit tests separately (they might take a long time) or if you wanted to publish your artifact as a dependency (i.e. manual task to push the artifact which is created in your other configuration). 

### GitVersion

We are using this as a meta-runner within TeamCity, there are other options as to how you use this tool such as command line or MSBuild tasks. What this does is create our SemVer or NuGet version automatically for us based on our git history. Previously we were doing a lot of the versioning manually and then we came up with a powershell script to achieve a similar outcome until we stumbled upon this which does it all for us. It follows Gitflow and Github flow conventions and can be applied to other branching strategies with some configuration.  

## Octopus Deploy

Used to deploy an artifact (NuGet package) to a given environment, as well as setting up a website in IIS and updating variables for different SQL connections. Our process here is largely the same, yet Octopus Deploy has grown up a little since then and now has support for lifecycles, automatic release creation and (in next release) offline deployments amongst some other really useful features.

1. Test SQL connection
2. Grab NuGet package
3. Create / update website in IIS
4. Deploy files to website folder
5. Update variables / apply transforms
6. Test URL (200 OK)
7. Notify status of deployment via Slack
8. Clean up / apply retention policies

Steps 2, 3, 4 and 5 can actually be done via one step in Octopus Deploy (deploy a NuGet package), but I have split it out here for better readability. We have also added some basic tests around our deployment...

- Test SQL connection : if we can't access the database using the provided connection string, we don't deploy
- Test URL : we ensure that we get a 200 OK status back once we have deployed
- Notify status : we use Slack for sending out a deployment status (could also send an email if you prefer traditional methods)

Depending on the project or usage case you might want to do other steps such as backup the database or website folder, grant permissions to a certain user, or install a package required for deployment via Chocolatey for example. There are loads of other options in the step template library: [https://library.octopusdeploy.com/](https://library.octopusdeploy.com/)

### Lifecycles

This was introduced in 2.6 and allowed for structuring your deployment process between environments. So, you could set it up to no longer allow a release package to be deployed straight to Production without any testing for example. This forces you to take a release through the proper deployment process and get sign-off before promoting.

So, an example lifecycle could be set up like this...

**Internal testing**

- Dev (auto-deploy)
- QA

**Client testing** (any 1)

- UAT
- Pre Prod

**Go live** (any 1)

- Production
- DR (backup servers)

Within this you can specify whether all of the environments need to be deployed to or at least one in the lifecycle phase, denoted in brackets above. You can also set different retention policies per phase, so you would probably want to keep all releases in the 'go live' phase, but maybe the latest 5 in the 'internal testing' and 'client testing' phase. 

### 3.0

A [new version](http://octopusdeploy.com/blog/octopus-3.0-pre-release-is-here) of Octopus Deploy is in pre-release. With this comes a bunch of changes and new features...

- Deployment targets which allow for offline deployments, Azure websites, SSH and Linux support
- Rebuilt with SQL Server rather than RavenDB 
- Improved performance
- Something called [delta compression](http://octopusdeploy.com/blog/the-octopus-deploy-3.0-time-saver-delta-compression), which only transfers things that have changed in your package and should make deployments a lot quicker. 
- Migration tool so you can export your configuration into JSON and import into other instances of Octopus Deploy
- Changes to tentacle architecture, which means that the deployment aspect of a tentacle is no longer tightly coupled to the Octopus version. Enter Calamari, a command-line tool invoked by Tentacle during a deployment for doing deployment tasks. It is also [open-source](https://github.com/OctopusDeploy/Calamari), so you can fork and make it your own.