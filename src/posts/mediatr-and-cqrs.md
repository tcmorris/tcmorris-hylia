---
title: Mediatr and CQRS
metaDesc: How to configure Mediatr to help apply CQRS in your project.
date: '2020-01-30'
tags: 
- dotnet
---

## Introducing new changes

To begin, I think it would be useful to go over a scenario that I'm sure many people have come across before and why perhaps it would be useful to look into an alternative. 

Let's say we've got a bunch of beers and we've allowed our users to see them along with their basic details and an option to buy. When someone is interested in that beer, they might want to know whether or not this beer is any good. It'd be great if we could allow our users to review a beer. 

When using the repository pattern we would have created a new repository, passed that into a service and called that service from within our controller to return the reviews and allow our users to review a beer. We will have likely used the same model for our read and write. This works and you carry on as normal happy that users can now add some extra context to beers. 

```csharp
public class BeerReviewService
{
    private readonly Database _db => ApplicationContext.Current.DatabaseContext.Database;

    public void Create(BeerReview beerReview)
    {
        _db.Insert(beerReview);
    }

    public IList<BeerReview> GetAllReviews()
    {
        return _db.Fetch<BeerReview>("SELECT * FROM BeerReviews");
    }

    public IList<BeerReview> GetReviewsByBeerId(int beerId)
    {
        return _db.Fetch<BeerReview>("SELECT * FROM BeerReviews WHERE BeerId = @0", beerId);
    }
}
```

A couple weeks later, someone has started taking advantage of this rating option and posted a bunch of fake reviews. You need to delete them. Something that when you first built your service, the functionality wasn't required and for good reason didn't add it in just for the hell of it. Except that now, you need to make a change to your rating service by adding some more code. A couple more weeks, and people are asking to edit those reviews as they've added some typos. You need to introduce edit functionality and the rating service grows some more. Whilst we've now got some honest reviews, not everyone agrees with them and we'd like to allow people to upvote a review to show appreciation for how good a review is. Again, we need to go back to our rating service. This can go on and on and soon enough the original code that you wrote bares little resemblance to the intentions of what was supposedly a simple implementation. The contract itself has completely shifted.

```csharp
public class BeerReviewService
{
    private readonly Database _db => ApplicationContext.Current.DatabaseContext.Database;

    public void Create(BeerReview beerReview)
    {
        _db.Insert(beerReview);
    }

    public void Edit(BeerReview beerReview)
    {
        _db.Save(beerReview);
    }

    public void Upvote(int reviewId)
    {
        var beerReview = GetReviewById(reviewId);
        if (beerReview == null)
        {
            return;
        }

        beerReview.UpVotes += 1;
        _db.Save(beerReview);
    }

    public void Delete(BeerReview beerReview)
    {
        if (beerReview == null)
        {
            return;
        }
        _db.Delete(beerReview);
    }

    public IList<BeerReview> GetAllReviews()
    {
        return _db.Fetch<BeerReview>("SELECT * FROM BeerReviews");
    }

    public IList<BeerReview> GetReviewsByBeerId(int beerId)
    {
        return _db.Fetch<BeerReview>("SELECT * FROM BeerReviews WHERE BeerId = @0", beerId);
    }

    public IList<BeerReview> GetReviewsByUserId(int userId)
    {
        return _db.Fetch<BeerReview>("SELECT * FROM BeerReviews WHERE UserId = @0", userId);
    }

    public BeerReview GetReviewById(int reviewId)
    {
        return _db.FirstOrDefault<BeerReview>("SELECT * FROM BeerReviews WHERE ReviewId = @0", reviewId);
    }
}
```

## How about CQRS?

Taking a look at our reviews example, there were a few things we ended up wanting to do, but essentially we can split those into something that modifies something, or something that retrieves something. 

[CQRS](https://docs.microsoft.com/en-us/azure/architecture/patterns/cqrs) or Command Query Responsibility Segregation in it's full name, is a pattern that separates command logic from query logic. If you need to make an update or create an entity, you would use a command. If you need to query for some data, you would create a query. By following this pattern, it can help with the Single Responsibility Principle. No longer are you likely to have a service or such like that combines both update and retrieval, providing a better separation of concerns. GraphQL has a [similar concept](https://graphql.org/learn/queries/), you can either `query` data or you can apply a `mutation` to change data.

One of the benefits of this is that the code itself should be self documentating. The name itself should provide all you need to know. For example, `CreateBeerReviewCommand` will as one might assume, create a review for a beer.

```csharp
public class CreateBeerReviewCommand : IRequest<int>
{
    public int BeerId { get; set; }
    public int UserId { get; set; }
    public string Review { get; set; }
}
```

As for a query, likewise the naming convention should be self explanatory. `GetBeerReviewsQuery` in order to fetch a collection of reviews for a beer. Things have suddenly become a lot more descriptive and relate better to their function. 

```csharp
public class GetBeerReviewsQuery : IRequest<BeerReviewsModel>
{
    public int BeerId { get; set; }
}
```

Once you've got a command or a query, you need to handle that request. One way to do that, is to create a new handler which mimics the naming convention of your command (or query). So, you will have a `CreateBeerReviewCommand` and a `CreateBeerReviewHandler` which will perform any necessary logic (i.e. create a beer review) and return a response. You define the expected response, so in the `CreateBeerReviewHandler` we will return the id of the newly created review for example.

```csharp
public class CreateBeerReviewHandler : IRequestHandler<CreateBeerReviewCommand, int>
{
    private readonly Database _db => ApplicationContext.Current.DatabaseContext.Database;

    public int Handle(CreateBeerReviewCommand request)
    {
        var beerReview = new BeerReview()
        {
            BeerId = request.BeerId,
            UserId = request.UserId,
            Review = request.Review,
            LastUpdated = DateTime.Now
        };
        _db.Save(beerReview);

        return beerReview.ReviewId;
    }
}
```

## An example

If we go back to our earlier example, when asked to allow users to edit their reviews, we can implement a new `EditBeerReviewCommand` with it's own handler and it's own logic. 

```csharp
public class EditBeerReviewCommand : IRequest
{
    public int ReviewId { get; set; }
    public string Review { get; set; }
}

public class EditBeerReviewHandler : IRequestHandler<EditBeerReviewCommand>
{
    private readonly Database _db => ApplicationContext.Current.DatabaseContext.Database;

    public void Handle(EditBeerReviewCommand request)
    {
        var beerReview = _db.GetById(request.ReviewId);
        if (beerReview != null)
        {
            beerReview.Review = request.Review;
            beerReview.LastUpdated = DateTime.Now;
            _db.Save(beerReview);
        }
    }
}
```

You'll notice that the code being used to edit a beer review only needs to reference the review id and provide the newly updated review itself. The models in this sense are a lot smaller than passing around a DB model, and they are created to perform a particular task. You do end up with more, but hopefully you can see the benefit in doing so.

Now, you might be thinking we've got a ton of classes going on here and I'm going to have a lot of dependencies in my controller to achieve all of this. You need to ensure the right commands/queries are going to the right handlers, it would be good to have something that could help mediate these dependencies. Luckily there is a Nuget package out there that makes things a bit simpler. Enter **Mediatr** by Jimmy Bogard (of AutoMapper fame).

[Mediatr](https://github.com/jbogard/MediatR) is a package that helps link up the relevant commands/queries to the correct handlers. This means that we can simplify our controllers to use an instance of `IMediatr`, which we'll then use to send our request to get the relevant response. In the example below, we've got a beer page and we're getting the reviews for that beer by creating a new `GetBeerReviewsQuery`.

```csharp
public class BeerController : Controller
{
    private readonly IMediator _mediatr;

    public BeerController(IMediator mediatr)
    {
        _mediatr = mediatr;
    }

    public IActionResult Index()
    {
        var beerId = 1234;
        var model = new BeerViewModel
        {
            BeerId = beerId,
            Name = "My beer",
            Description = "My awesome pale ale",
            Price = 2.99m,
            Reviews = _mediatr.Send(new GetBeerReviewsQuery { BeerId = beerId })
        };

        return View(model);
    }
}
```

In this scenario we've isolated the code to get reviews relating to a beer and we're loading them our handler for `GetBeerReviewsQuery`. It'll look something like this.

```csharp
public class GetBeerReviewsHandler : IRequestHandler<GetBeerReviewsQuery, BeerReviewsModel>
{
    private readonly Database _db => ApplicationContext.Current.DatabaseContext.Database;

    public BeerReviewsModel Handle(GetBeerReviewsQuery request)
    {
        var reviews = _db.Fetch<BeerReview>("SELECT * FROM BeerReviews WHERE BeerId = @0", request.BeerId);
        var response = new BeerReviewsModel()
        {
            Reviews = reviews,
            Count = reviews.Count,
            LastUpdated = DateTime.Now
        };        

        return response;
    }
}
```

## Vertical Slices

Now that we've split our application more closely into commands and queries relating to specific tasks, it might be a good idea to group those tasks together somehow. This technique involves organising your project into vertical slices rather than by type. i.e. we can define features and split them into verticals. You can keep code relating to beer reviews as one slice of your application, rather than simply having controllers, services, models and so on with a combination of multiple domains. When you need to modify the beer reviews code, you know where to find it and the relating code is positioned together as one piece. You may wish to use different projects or release as different features in a microservices architecture, in which case this idea would help with providing updates in isolation. From a testing point of view, the easier to read and smaller classes should help resulting in code that should be more maintainable as the project grows.

## Going further

The examples provided here are relatively basic to show the concepts and how you might work with this pattern, however in a typical world you'll also need to think and caching, validation, logging and various other steps in the pipeline. Thankfully, Mediatr has the concept of pipelines and adding behaviours, so you can run code before or after handlers. I'll leave that for another time.

### References

- [https://codeopinion.com/fat-controller-cqrs-diet/](https://codeopinion.com/fat-controller-cqrs-diet/) 
- [https://medium.com/@dbottiau/a-naive-introduction-to-cqrs-in-c-9d0d99cd2d54](https://medium.com/@dbottiau/a-naive-introduction-to-cqrs-in-c-9d0d99cd2d54)
- [https://www.stevejgordon.co.uk/cqrs-using-mediatr-asp-net-core](https://www.stevejgordon.co.uk/cqrs-using-mediatr-asp-net-core)
