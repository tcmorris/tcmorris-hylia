---
title: Updating older Umbraco packages
metaDesc: A collection of packages / data types and their equivalents in Umbraco 7+.
date: '2016-10-27'
tags: 
- umbraco
---

In a [recent post](http://tcmorris.net/blog/upgrading-umbraco/), I outlined some of the reasons for upgrading Umbraco and how best to approach the challenge. In that post, I mentioned that you are likely to have a number of issues with custom packages or custom data types. Below is a rundown of some of the examples I have encountered. If you have found others, it'd be great if you could add to the comments section and I'll look to edit this post with extra details in the hope that it can become a valuable resource.

### Packages

This is basically a list of packages which I have come across via a previous upgrade to Umbraco 7+. Quite a few will have a version that works with Umbraco 7+, others will simply not work. Where possible, I have included a suitable upgrade path.


| Package                   | Description                                                                                                                   | v7 support?      | Alternative                    |
|---------------------------|-------------------------------------------------------------------------------------------------------------------------------|------------------|--------------------------------|
| AttackMonkey Custom Menus | used to disable delete/copy/move etc on some content nodes, to idiot proof the CMS (e.g. Stop users deleting the home page)   | Yes (v2+)        |                                |
| AttackMonkey Tab Hider    | used to hide some of the tabs on content types from certain users (e.g. Only admins can edit the SEO tabs)                    | No               |                                |
| AttackMonkey Security     | security helper, password strength validation                                                                                 | No               | Membership provider regex      |
| AutoFolders               | used to automatically organise news type content into Year/Month folders                                                      | No               | DateFolders / uDateFoldersy    |
| CMSImport                 | used to import content from an old CMS                                                                                        | Yes              |                                |
| Config Tree               | allows you to view all the site config files in the back office                                                               | Yes              |                                |
| Contour                   | used to provide forms                                                                                                         | Yes (use latest) |                                |
| Contour Contrib           | adds some additional functionality to Contour, e.g. Recaptcha                                                                 | Yes              |                                |
| DAMP                      | used as a replacement for the built in media picker, as it offers more functionality                                          | No               | Built in                       |
| DocType Mixins            | plugin to allow DocType composition in earlier versions of Umbraco                                                            | Sort of          | Doc Type Compositions          |
| Embedded Content          | allows for repeating content structures within a single page                                                                  | No               | Archetype / Nested Content     |
| FamFamFam icons           | adds additional icons for use in the content tree to represent Document Types                                                 | No               | Built in / packages            |
| Google Maps for Umbraco   | Data Type that allows the selection and rendering of Google Maps                                                              | No               | Alternative packages available |
| ImageGen                  | allows image resizing and compression                                                                                         | Yes              | Image Processor                |
| Mass Replacer             | bulk find and replace actions for Umbraco back office, occasionally used when site wide brand names need standardising etc    | Maybe            |                                |
| Media Icons               | displays file type icon rather than the built in ones in the Umbraco media library                                            | No               |                                |
| Open Calais Autotag       | can be used as a replacement for the tags data type in earlier versions of Umbraco                                            | No               |                                |
| Path Fixup                | This is a developer dashboard control to fix database issues                                                                  | N/a              | Fixed                          |
| Repeatable Custom Content | A datatype which allows adding repeatable custom contents/child nodes.                                                        | No               | Archetype / Nested Content     |
| Robots.txt                | allows you to view the robots.txt file for the site in the back office                                                        | Yes              |                                |
| Structure Extensions      | allows you to set default Document Types for child pages                                                                      | Maybe            | Built in                       |
| uComponents               | used for various additional Data Types                                                                                        | No               | Built in                       |
| Yoyo CMS Tag Manager      | a plugin that adds an additional section to the CMS allowing you to visually view and manage all of the tags used on the site | Yes (v3+)        |                                |


### Data Types

Ok, so this one is more of an extension to the packages section above, but I figured it'd be useful to separate the two.


| Property Editor               | v7 support? | Alternative           | Conversion  |
|-------------------------------|-------------|-----------------------|-------------|
| DAMP                          | No          | Built-in media picker |             |
| Embedded Content              | No          | Nested Content        | Manual      |
| Form Picker                   | Yes         |                       |             |
| Google Map                    | No          | Package               |             |
| Legacy MNTP                   | No          | MNTP                  | XML to CSV  |
| Short URL Field               | No          | Textbox?              |             |
| uComponents: Multiple Dates   | No          | Package               | Manual      |
| uComponents: Multi-URL Picker | No          | Related Links         | Manual      |
| uComponents: URL Picker       | No          | URL Picker            | XML to JSON |
| XPath DropDownList            | No          | MNTP                  |             |


As mentioned before, this is by no means a complete list but should offer some guidance for others who come across a similar task. You might have an existing site or customer who uses some of these packages and been holding off an upgrade. It's easy enough to figure out what the outcome of such an upgrade will be, so I'd say give it a go and figure out what breaks (if anything).