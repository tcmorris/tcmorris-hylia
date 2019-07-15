---
title: Using migrations in Umbraco
metaDesc: How to set up migrations for your Umbraco project.
date: '2018-10-28'
tags: 
- umbraco
---

Migrations are a really handy way to deploy your database changes to new environments. Umbraco use them extensively for processing any upgrades between versions and you can use them too. If you've ever used Entity Framework, then this will probably be fairly familiar.

# What does it do?

Umbraco has actually been doing this since v6, with the idea that running migrations via code versus having a suite of custom SQL scripts can offer more options. 

There is a table which keeps track of all the migrations that have been run called `umbracoMigration`, it looks a bit like this:

- add image

You'll notice that it has a name next to each migration, as well as the version. This is how we can see what state our database is in. So, for the above it's run up to 7.12.3 in Umbraco. When you deploy changes to a new server, if it's pointing to an older version then it will run the migration on startup. This is mostly used for schema changes, but you can also insert data or run your own custom code.

### How does it work?

Umbraco has this concept of a MigrationRunner that you can trigger to execute your migrations. You'll need to set the target version and then you can find all the relevant migrations to get to that version and run them.

In order to find these migrations, all you need to do is create a file that looks a bit like this:

```csharp
/// <summary>
/// MyCustomTable migration
/// </summary>
[Migration("1.0.0", 1, MigrationNames.MyCustomNamespace)]
public class MyCustomTableMigration : MigrationBase
{
    private const string TableName = "MyCustomTable";

    public MyCustomTableMigration(ISqlSyntaxProvider sqlSyntax, ILogger logger)
        : base(sqlSyntax, logger)
    {            
    }

    /// <summary>
    /// Process database upgrade
    /// </summary>
    public override void Up()
    {            
        // create a new table if it doesn't exist
        var tables = SqlSyntax.GetTablesInSchema(Context.Database).ToList();
        if (!tables.InvariantContains(TableName))
        {
            Create.Table(TableName);
        }

        // or you can run alterations on existing tables
        var columns = SqlSyntax.GetColumnsInSchema(Context.Database).ToArray();
        var columnExists = columns.Any(x =>
            string.Equals(x.TableName, tableName) &&
            string.Equals(x.ColumnName, columnName)
        );
        if (!columnExists)
        {
            Alter.Table("SomeOtherTable").AddColumn("MyCustomString").AsString().Nullable();
        }
    }

    /// <summary>
    /// Process database downgrade
    /// </summary>
    public override void Down()
    {
        // drop the table
        Delete.Table(TableName);

        // remove the column from existing table
        Delete.Column("MyCustomString").FromTable("SomeOtherTable");
    }
}
```

The code above is saying add a new table and also alter an existing table by adding a new column of type string. The commands are exposed via the inherited `MigrationBase` which Umbraco provides. We're stating that this is v1.0.0 in a SemVer format. We're also saying that this is going to have it's own name for the migration (`MigrationNames.MyCustomNamespace`), this keeps all your migrations grouped for your custom code.

We need to tell Umbraco to run this code on startup. Here is how you can do that:

```csharp
public class CustomMigrationEventHandler : ApplicationEventHandler
{
    protected override void ApplicationStarted(UmbracoApplicationBase umbracoApplication, ApplicationContext applicationContext)
    {
        // check target version
        var rawTargetVersion = ConfigurationManager.AppSettings["app:MigrationVersion"] ?? "1.0.0";
        var targetVersion = SemVersion.Parse(rawTargetVersion);
        if (targetVersion != null)
        {
            HandleMigrations(targetVersion);
        }

        base.ApplicationStarted(umbracoApplication, applicationContext);
    }

    private void HandleMigrations(SemVersion targetVersion)
    {
        // get all migrations already executed
        var currentVersion = new SemVersion(0, 0, 0);
        var migrations = ApplicationContext.Current.Services.MigrationEntryService.GetAll(MigrationNames.MyCustomNamespace);

        // get the latest migration executed
        var latestMigration = migrations.OrderByDescending(x => x.Version).FirstOrDefault();
        if (latestMigration != null)
        {
            currentVersion = latestMigration.Version;
        }

        if (targetVersion == currentVersion)
        {
            // nothing to do
            return;
        }

        var migrationsRunner = new MigrationRunner(
            ApplicationContext.Current.Services.MigrationEntryService,
            ApplicationContext.Current.ProfilingLogger.Logger,
            currentVersion,
            targetVersion,
            MigrationNames.MyCustomNamespace);

        try
        {
            // run migrations
            migrationsRunner.Execute(UmbracoContext.Current.Application.DatabaseContext.Database);
        }
        catch (Exception e)
        {
            LogHelper.Error<MigrationEvents>("Error running migration", e);
        }
    }
}
```

That's a fair chunk of code. Quick run through as to what's happening...
- find out what version to target
- figure out if that is a later version that what we have
- if so, run the migrations
- throw an error if it blows up

Your custom table should get added the next time you start Umbraco. Notice that I've added an appsetting for keeping track of the version called `app:MigrationVersion`. You can add this via your web.config and then use it to target different versions and update through your deployment variables. This way, we opt in and have a configurable value rather than changing our code each time.

So, we've added our custom table and now want to make a change. What do we do? Well, we create a new migration which inherits from `MigrationBase`, has our code in and then bump up the version via the config value.

### Any gotchas?

Yeah, a few things to be aware of. 

In the migration examples above, we used static strings to denote the table and column names. You can also use a reference like so, which uses type `T`.

```csharp
Create.Table<MyCustomTable>();
```

The slight issue with this though is that if you want to remove `MyCustomTable` from your code, then you're going to have to change your migration as you don't particularly want to keep old references around just so that it will compile. The migration shouldn't change though, it should be able to run multiple times and by the end you should be in the same state as the version asked for. The term for this is idempotent.

The other thing that to note is that once you've racked up a few of these, it might be hard to realise which ones relate to which version. Umbraco handles this by using named folders that relate to the version they are targetting. Here's an alternative:

- 001_InitialMigration.cs
- 002_MyNextMigration.cs
- 003_AddCustomerTable.cs

Essentially, just adding a prefix so that you can easily see at a glance which migrations have been added and what order they occurred. You could also map these to the SemVer version if you wanted.

When you deploy these changes to your site, they will run on startup and could provide a bit of downtime for your customers. Think about what that means to you and figure out if you can do this at low traffic or you could use a blue-green strategy where you alternate between 2 versions of the live site allowing you to prep everything without your customers taking the hit.

### Summary

Hopefully you now understand a little about what migrations are and can see how to utilise them on your projects. Following the patterns mentioned in this post, you should be deploying with ease.

#### References

- [https://cultiv.nl/blog/using-umbraco-migrations-to-deploy-changes/](https://cultiv.nl/blog/using-umbraco-migrations-to-deploy-changes/)
- [https://github.com/umbraco/Umbraco-CMS/tree/dev-v7/src/Umbraco.Core/Persistence/Migrations/Upgrades](https://github.com/umbraco/Umbraco-CMS/tree/dev-v7/src/Umbraco.Core/Persistence/Migrations/Upgrades)
- [https://github.com/umbraco/Umbraco-CMS/blob/dev-v7/src/Umbraco.Tests/Migrations/CreateTableMigrationTests.cs](https://github.com/umbraco/Umbraco-CMS/blob/dev-v7/src/Umbraco.Tests/Migrations/CreateTableMigrationTests.cs)
- [https://skrift.io/articles/archive/umbraco-migrations-made-easy/](https://skrift.io/articles/archive/umbraco-migrations-made-easy/)
