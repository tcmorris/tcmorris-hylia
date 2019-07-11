---
title: Courier investigation
tagline: A look at Courier for Umbraco 7 including installation and basic usage guidelines.
tags: umbraco
---

### Installation

Download from here: [https://our.umbraco.org/projects/umbraco-pro/umbraco-courier-2](https://our.umbraco.org/projects/umbraco-pro/umbraco-courier-2)

- Go to required instance of Umbraco to install on.
- Go to Developer tab
- Open up packages folder and select Install local package
- Choose downloaded zip file
- Confirm install
- Wait for Umbraco to reload
- Add location(s) you would like to promote to

**Example location**

This is configurable in *courier.config*

    <repository name="Example QA site" alias="example-qa" type="CourierWebserviceRepositoryProvider" visible="true">
      <url>http://test.client.example.co.uk/</url>
      <user>0</user>
    </repository>

Courier works on the basis of having a connection between Umbraco instances and then being able to compare and push changes. If there is no connection for Umbraco, then a sync will not be possible. This also means that Courier needs to be installed on all environments that need to be in sync.

### Usage

Once the locations have been set up and proven to work. i.e. they are able to connect without error, the usage from a client perspective is rather simple.

**Basic usage**

There will be a new context-sensitive option when selecting content called Courier. When this is selected a new window will open asking for a target machine to deploy to, along with confirmation of what content will be transferred. Click the button to deploy and wait for Courier to package the content and transfer across to the other location.

**Revisions**

An alternative to using the context-sensitive option is to go into the newly added Courier section where you can add a revision. This allows you to choose more than just content and let's you choose things like Dictionary items, Document types, Languages, Media among others. So, if you had created a bunch of content that had some media and potentially some files you wanted to deploy, you could create a revision to transfer elsewhere. The idea is that whatever needs to be transferred, can be. Courier will also help you out and sort out the dependencies if there are any. Once you have selected your options and created your revision (package), then this will be available on other locations to transfer.

**Configuration**

Since Courier has is it's own section, there is the option to restrict access to revisions. It is possible that you may only want to give developers access to this section whilst giving clients access to the basic use of selecting content. If you do wish to give clients access to the Courier section, then you can also choose what you want them to be able to transfer. So, you probably wouldn't want them to know about the Datatypes or Document types, but you would probably want them to be able to transfer Files, Folders and Media.

By default, Courier is set up quite nicely and will cover most usage cases however there are some other configuration options that might be preferable.

Choose which folders can be included in revisions.

	<folderItemProvider>
      <include>
        <!--<folder>~/media/assets/somefolder</folder>-->
      </include>
    </folderItemProvider>

Choose which files can be included in revisions.

	<fileItemProvider>
      <!--<folder>~/media/assets/somefolder</folder>-->
      <!--<file>~/media/assets/somefile.png</file>-->
    </fileItemProvider>

Allow for children/parent media to be included.

	<mediaItemProvider>
      <includeChildren>false</includeChildren>
      <includeParents>true</includeParents>
    </mediaItemProvider>

Allow/deny access by IP/users. (in security element)

	<filters>
      <ipfilter>
        <allow>*</allow>
      </ipfilter>
      <userfilter>
        <allow>*</allow>
        <!--<deny>editor</deny>-->
      </userfilter>
    </filters>
