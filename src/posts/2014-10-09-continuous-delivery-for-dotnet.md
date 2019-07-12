---
title: Continuous delivery for .NET
tagline: A look at using TeamCity and Octopus Deploy to package up and deploy applications.
tags: 
- teamcity 
- octopusdeploy 
- dotnet
---

One very important factor of being a developer is deploying your code to the appropriate environment without anything failing. In order to do this, we should be automating as many tasks as possible to reduce human failure. Of course, some degree of human interaction should happen but by in large, we shouldn't need to do much once we have set up our deployment process.

## Simple guide ##

At it's very basic, this should happen every time we work on a project. 

1. Create project + repo
2. Write some code
3. Push some code to repo
4. Grab code and push to server

Now... parts 1, 2 and 3 will largely be the same wherever, but 4 could be done a number of different ways and there are various different settings / updates required to get a working website. Let's not go into too much details of the different options, but look at a particular method for making this work.

## TeamCity and Octopus Deploy ##

In order to for us to deploy, we are going to use TeamCity and Octopus Deploy. There is a bit of configuration required to do so, but this should be simpler than the traditional Web Deploy method.

It is worth noting that one of the goals in deployment is to allow different roles within the process. So, the devs can push code and then the sysadmins or tech leads can manage deployments. Therefore, publish from Visual Studio is completely out of the question. This has a few advantages in that the relevant people are in charge of their remit. For example, a dev doesn't have the ability to push changes to Production and therefore a degree of protection / sign-off is involved. sysadmins can create a new server and see at a granular level what gets pushed if they choose to. Important server details are not exposed, which can be the case with Web Deploy.

Below is the basic overview of steps required in set up...

1. Install TeamCity on build server (one time only)
2. Install [Octopus TeamCity plugin](http://docs.octopusdeploy.com/display/OD/TeamCity) (one time only)
3. Install [Octopus Server](http://docs.octopusdeploy.com/display/OD/Installing+Octopus) on build server (one time only)
4. Create servers for project (one time only for each project)
5. Install [Octopus Tentacle](http://docs.octopusdeploy.com/display/OD/Installing+Tentacles) on required servers (one time only for each server)
6. Install [OctoPack](http://docs.octopusdeploy.com/display/OD/Using+OctoPack) in project via NuGet (one time only for each project)
 
As shown, a lot of these tasks you will not need to revisit and once completed, will allow you to deploy without the need for manual intervention. 

**TeamCity**

Used to build the solution, perform tests and create an artifact for deployment. This will have continuous integration set up for the dev branch so that we don't have to trigger the release each time. We can use a template to share this across projects.

1. Checkout repository and build solution (run OctoPack)
2. Perform tests and check code coverage
3. Publish package to Octopus NuGet server
4. Create Octopus release (trigger Octopus Deploy)

**Octopus Deploy**

Used to deploy an artifact to a given environment, as well as setting up a website in IIS and updating variables for different SQL connections. This all happens on the server we are deploying to via a secure connection.

1. Grab NuGet package
2. Create website in IIS
3. Deploy files to website folder
4. Update variables / apply transforms
5. Clean up / apply retention policies

When a release is created, Octopus Deploy will keep this indefinitely unless you tell it to. This means that you can rollback quickly to a prior release, but also means that you could be left with a number of files left on the server that you don't want. Therefore, within Octopus Portal you can amend the number of releases you want to keep and for how many days. More info can be found in the [documentation](http://docs.octopusdeploy.com/display/OD/Retention+policies).

**Why use both?**

It is true that you could use TeamCity to deploy to each environment and perform configurations for you. However, this is reliant on Web Deploy being installed on the server you wish to deploy to. This is not a requirement for Octopus Deploy, which uses the idea of tentacles to open up a communication between the build server and the web server. This has several [security enhancements](http://docs.octopusdeploy.com/pages/viewpage.action?pageId=360622) and is generally easier to configure. You could also get Octopus Deploy to set up a site in IIS or perform Powershell tasks for you, something TeamCity and other build platforms are not built for. 

All in all, it is about using what is built for the task at hand. TeamCity can be used for building the code, running the tests and creating a single release package. Octopus Deploy can then be used to deploy this NuGet package wherever you want and ensure that the configuration is correct for the environment, even cleaning up files in the process.

If you would like to read more, there is a [blog post](http://octopusdeploy.com/blog/octopus-vs-build-server) detailing this approach, which explains why Octopus Deploy was created.

## Example setup ##

- 1 build server with TeamCity and Octopus server installed
- 1 web/db server for internal Development/QA (shared web and db)
- 1 web/db server for UAT (could be separate web and db)
- 1 web/db server for Production (could be separate web and db)

In your project you would need the following **transforms**...

- Web.Development.config
- Web.QA.config
- Web.UAT.config
- Web.Production.config

These will get run automatically when uploaded to the web server by Octopus Deploy. There are a couple of things to take note of, however. Ensure that the transform matches the environment name in Octopus Deploy and make sure that the files are included in the NuGet package. For more notes on this, check the [documentation](http://docs.octopusdeploy.com/display/OD/Configuration+files).

Octopus Deploy also has this idea of **variables** that can be set within the Octopus Portal (admin area). This means that you can define your variables and specify which environment these will apply to. As long as you have these turned on for your project, they will overwrite the name/value pairs within your config files (not just web.config). There is also the added benefit of being able to define variable sets and inherit these on a project. One such example, could be a set of logging options that you only enable for UAT and Production, but you tend to do this on every project. With a variable set, you can ensure they are included with little configuration. See the [documentation](http://docs.octopusdeploy.com/display/OD/Variables) for more examples.
