---
title: Building a bottle shop
metaDesc: How I built a custom store in Umbraco using .NET, VueJS and Algolia.
date: '2020-08-10'
tags: 
- umbraco
- frontend
- projects
---

One of the things that is important to an ecommerce experience is the speed at which you can make informed decisions and place items in your cart. From there, the next phase will be to checkout and proceed with the order. Typically, this is classed as a conversion and you want as many as possible. You might do various tests to achieve that end goal and nudge the customer along that path.

What if you were open to the idea of not pushing the customer to checkout immediately and provide them time to make their choices, even over a few weeks. This was a concept that we played with at [BeerBods](https://beerbods.co.uk/), centred around a reservation of beer, to build a case of beers you pick in your own time. The experience itself is more about exploration and finding the beer you want.

Not only does that shift the UX in a slightly different direction, it also has some technical challenges. Namely, how do you reserve a beer, how is this stock controlled and how do I come back to my continue with my purchase. 

<img src="/images/buildacase-concept.jpg" title="Build a Case - concept" />

## Introducing Umbraco

[Umbraco](https://umbraco.com/) as a CMS allows for a lot of variation in terms of document types and property editors. Essentially, you can build a custom editing interface relatively easily to manage your content. In my case, I wanted to add products to Umbraco and have these editable by anyone in the team. There was no data warehouse, so adding directly into Umbraco was ok in this scenario. I was able to define the required information and set up some basic stock control with SKUs and pricing. Since we were already using Umbraco, it was comfortable to use on this front. 

The product pages themselves can be rendered using traditional patterns and templates. The more interesting part came when adding the functionality to add a beer to your case.

## Enter VueJS and Algolia

[VueJS](https://vuejs.org/) is a javascript framework that allows for building SPA (single page applications) like functionality, introducing more dynamic and responsive user interfaces. It can be added with relative ease to an existing website, which made it a good option for building our 'shop' front. VueJS would be responsible for rendering components, handling FE state and making calls to our API. 

[Algolia](https://www.algolia.com/) is a search platform that has a number of pre-built components to integrate with it's service. There was one of particular interest that near enough provided the basis for our ['shop'](https://www.algolia.com/doc/guides/building-search-ui/widgets/showcase/vue/) and the required functionality. This included a list of results, options to filter, sort and search. The idea of adding this kind of service had a number of benefits, it was quick to get started and by having an external service handling search powered by a CDN, it was very responsive to customers. We could push data to the service (whenever a product was updated) and query the results wherever we needed.

Combined, we were able to build a UI that was quick to load, easy to filter and visually interesting with a lot of customisation options. All of the API calls would be triggered from the FE and the state would update our UI to present back to the user.

## Adding Fluidity

So, we've got our products and our UI for the front end. We now need to persist changes and provide a stateful system for the current user. This was done by implementing custom database tables and an API to enable adding items, removing items and saving these items for a later date.

These custom database tables can be exposed in our Umbraco instance by using a package called [Fluidity](https://github.com/mattbrailsford/umbraco-fluidity/). It provides a CRUD like experience for the editors so that we can see what's being added and when, as well as allowing us to add beers on the behalf of a customer.

It's important to note that the idea of persistance is 2-fold, once when we are on the FE and we need to ensure state between sessions and also on the BE, where we need to actually store the data and reserve the beer. This distinction means that we don't get people adding multiple beers filling up stock until they commit to saving that data. At which point we've got their details and can follow up if necessary. 

When it comes to purchase or if the customer comes back to their choices, we can load these from the API and allow the customer to proceed. The items can be locked in and an order can be made just like any other checkout, allowing the next stage of fulfilment in the warehouse.

If the customer didn't come back (which did happen), then we'd prompt them to finish their case via email. If after the 30 days they hadn't made a purchase, then we would need to remove their reservation and put the beers back into the shop for other customers. This was done by using [Hangfire](https://www.hangfire.io/), a tool for running background tasks at desired times. 

## The result

What we were looking for was a responsive, easy to use 'shop' front for customers to pick beer and save until they were ready to make a purchase. We needed the ability to add beers to our shop and view details about what was in progress or had been ordered. By using a combination of Umbraco on the back end, and VueJS / Algolia on the front end, the result turned out how we wanted it to and we saw great customer interaction. 

_Note: This functionality is no longer live on the website but you can view the [introductory blog post](https://beerbods.co.uk/blog/introducing-buildacase/)._
