---
title: ASP.NET vNext and Mac OS X
tagline: A quick look at how to set up ASP.NET on a Mac.
image: /public/images/aspnet_vnext_1.jpg
tags: dotnet
---

One of the highlights of ASP.NET vNext is that it will be cross-platform. What this means is a few things...

- No more need for Windows VMs (designers will be happy)
- Potential for cheaper hosting, can run ASP.NET on Linux 
- Open to more developers (plus benefits of open source also)
- It's pretty cool.

### Installing DNVM / DNX

The first place to go is here: [https://github.com/aspnet/Home](https://github.com/aspnet/Home)

This will outline the ideas behind the next version of ASP.NET and offer some instructions for install. Below are the outlines.

<hr />

The key to getting set up is something called the .NET Version Manager (DNVM), which is a command line tool you use to download DNX (.NET execution environment). The DNX contains the code to bootstrap and run our applications. 

> A lot of this is built into the latest preview of Visual Studio 2015.

The steps to install DNVM on OS X are far easier if we have [Homebrew](http://brew.sh/) installed. So to get that we need to run:

	ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

To install the DNVM:

	brew tap aspnet/dnx
	brew update
	brew install dnvm
	
Then, we need to ensure that the command is registered with our bash profile:

	echo "source dnvm.sh" >> ~/.bashrc
	
You can run DNVM to get the latest version of DNX so that we can run our applications:
	
	dnvm upgrade
	
### Creating an app

Now we should have ASP.NET available to us on our Mac. The next thing we want to do is create a test application. To do this, we can use Yeoman to generate this for us. There are a few dependencies... so, to set these up we need to do some more command line entries.

Install node.js:

	brew install node

Install Yeoman:

	npm install -g yo
	
Install generator-aspnet

	npm install -g generator-aspnet

Details of the ASP.NET vNext generator are here: [https://github.com/OmniSharp/generator-aspnet](https://github.com/OmniSharp/generator-aspnet)

Now that we have the yo generator, we can type this into a command prompt:

	yo aspnet
	
... and what you should end up with is something that looks like this:

<img src="/public/images/aspnet_vnext_1.jpg" title="generator-aspnet on OS X" width="100%" />
	
Pretty cool huh?! In this example, I am going to choose to create a real basic console application, but the templates allow you to create web applications also.

After answering a few questions, we should have something that we can run and obtain the output from our test console application:

	dnu restore
	dnx . run
	
<img src="/public/images/aspnet_vnext_2.jpg" title="Running vNext on OS X" width="100%" />

And that just about sums up how to get started with ASP.NET vNext on OS X. 