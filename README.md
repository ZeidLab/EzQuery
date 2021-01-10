# EzQuery

## What is EzQuery?

 > __EzQuery__  is a  super easy to use, yet powerful .Net Core 5 library,
 to build SQL queries, to execute them , and to return populated sets of results of given
 table models. It does not contain a lot of features but it does the job.

# Table of Contents

[TableOfContents]:#table-of-Contents
[TableModel]:#what-is-a-table-model
[SupportedAnotations]:#what-are-the-supported-annotations
[UseAsBuilder]:#use-as-a-query-builder
[UseAsOrm]:#use-as-an-orm
[UseAsRepository]:#use-repository-pattern-helper-methods
[RepositoryMethods]:#ezQuery-repository-pattern-helper-methods
- [EzQuery](#ezquery)
  - [What is EzQuery?](#what-is-ezquery)
- [Table of Contents](#table-of-contents)
  - [How to install?](#how-to-install)
  - [How to use EzQuery?](#how-to-use-ezquery)
    - [Use as a query builder](#use-as-a-query-builder)
      - [Select](#select)
      - [Join](#join)
      - [Map To](#map-to)
      - [Order By](#order-by)
      - [Where](#where)
      - [Fetch Skip](#fetch-skip)
      - [Paged Results](#paged-results)
      - [Update](#update)
      - [Insert](#insert)
      - [Delete](#delete)
    - [Use as an ORM](#use-as-an-orm)
      - [Query Database](#query-database)
      - [Execute commands on Database](#execute-commands-on-database)
    - [Use repository pattern helper methods](#use-repository-pattern-helper-methods)
      - [Recommended folder structure](#recommended-folder-structure)
      - [Recommended DataContext class](#recommended-datacontext-class)
      - [Recommended repository class and it's interface](#recommended-repository-class-and-its-interface)
      - [EzQuery Repository Pattern Helper Methods](#ezquery-repository-pattern-helper-methods)
  - [What is a table model?](#what-is-a-table-model)
  - [Supported Component Model Data Annotations](#supported-component-model-data-annotations)
    - [What are the namespaces?](#what-are-the-namespaces)
    - [What are the supported annotations?](#what-are-the-supported-annotations)
      - [Table](#table)
      - [Key](#key)
      - [Column](#column)
      - [NotMapped](#notmapped)
      - [Required](#required)

  - [What is a table model?](#what-is-a-table-model)
  - [Supported Component Model Data Annotations](#supported-component-model-data-annotations)
    - [What are the namespaces?](#what-are-the-namespaces)
    - [What are the supported annotations?](#what-are-the-supported-annotations)
      - [Table](#table)
      - [Key](#key)
      - [Column](#column)
      - [NotMapped](#notmapped)
      - [Required](#required)

## How to install?

> Using **Nuget PackageManager**, you need to look for **ZeidLab.EzQuery**

> Using **Command line** : `Install-Package ZeidLab.EzQuery -ProjectName MyProject`

[^ Back To Top][TableOfContents]

## How to use EzQuery?

> There are 3 ways to use this Nuget Package , [As a Query Builder and ORM][UseAsBuilder]
> , [Just as an ORM][UseAsOrm] or [As a repository pattern][UseAsRepository].
> Of course you have the ability to choose any combination you need as well.


[^ Back To Top][TableOfContents]

### Use as a query builder

> To use the query building feature of EzQuery which is the most important and the main reason for building this package
> , you need to define a [Table Model][TableModel] first. Using
> `Microsoft.Data.SqlClient`
> , create an instance of `SqlConnection` and using below examples create your own SQL query.
> There are both Synchronous and Asynchronous methods to use.

[^ Back To Top][TableOfContents]

#### Select

> You can select all columns or just one column to return.
> Following code shows how to easily get all columns of Users table:

```csharp
    var queryBuilder = new EzQueryBuilder();
    await using (var conn = new SqlConnection(Settings.ConnectionString))
    {
        await queryBuilder.From<User>()
            .QuerySelectAsync(conn);           
    }
    List<User> users = queryBuilder.GetResult<User>();
```

> It is also possible to use Synchronously like this :

```csharp
    var queryBuilder = new EzQueryBuilder();
    using (var conn = new SqlConnection(Settings.ConnectionString))
    {
        queryBuilder.From<User>()
            .QuerySelect(conn);           
    }
    List<User> users = queryBuilder.GetResult<User>();
```

> You can even use it in one line like this :

```csharp
    using (var conn = new SqlConnection(Settings.ConnectionString))
    {
        List<User> users = new EzQueryBuilder()
                                .From<User>()
                                .QuerySelect(conn)
                                .GetResult<User>();           
    }
```

> You can get the value of specific properties by passing their name to `From<T>()`
> function like below example.In this example only Id and FirstName will be populated
> and every other properties value will be null. The whole User model will be returned
> as a result set.

```csharp
    var queryBuilder = new EzQueryBuilder();
    using (var conn = new SqlConnection(Settings.ConnectionString))
    {
        queryBuilder.From<User>(nameof(User.Id),nameof(User.FirstName))
            .QuerySelect(conn);           
    }
    List<User> users = queryBuilder.GetResult<User>();
```

[^ Back To Top][TableOfContents]

#### Join

> You can join and select from as many tables as you need. There is no limitation.
> But you need to
> Here is an example of Inner joining and selecting from Users and Profiles table
> `On [Users].[id] = [Profiles].[UserId]`

```csharp
    var queryBuilder = new EzQueryBuilder();
    await using (var conn = new SqlConnection(Settings.ConnectionString))
    {
        await queryBuilder.From<User>()
             .From<Profile>()
             .Join<User,Profile>(nameof(User.Id),nameof(Profile.UserId))
             .QuerySelectAsync(conn);           
    }
    List<User> users = queryBuilder.GetResult<User>();
    List<Profile> profiles = queryBuilder.GetResult<Profile>();
```

> We support `FULL JOIN, INNER JOIN, LEFT JOIN, RIGHT JOIN` and the default is **INNER JOIN**.
> The 3rd parameter is an ENUM and you can use it like below:

```csharp
    .Join<User,Profile>(nameof(User.Id),nameof(Profile.UserId),JoinTypes.INNER)

    .Join<User,Profile>(nameof(User.Id),nameof(Profile.UserId),JoinTypes.FULL)

    .Join<User,Profile>(nameof(User.Id),nameof(Profile.UserId),JoinTypes.LEFT)

    .Join<User,Profile>(nameof(User.Id),nameof(Profile.UserId),JoinTypes.RIGHT)
```

[^ Back To Top][TableOfContents]

#### Map To

> In the above Join Example, let's say we need only **Id,FirstName,LastName** from **Users**
> table and only **AvatarName** from **Profiles** table, and we want to get it as a single object.
> In this case we need to define an object with these properties and use mapper function to get those
> just like below example:

```csharp
    public class MyNewObject
    {
        public int Id { get; set; }
        public string Fname { get; set; }
        public string Lname { get; set; }
        public string AvName { get; set; }
    }
```

> And this is how we use it:

```csharp
    var queryBuilder = new EzQueryBuilder();
    await using (var conn = new SqlConnection(Settings.ConnectionString))
    {
        await queryBuilder.From<User>()
                .From<Profile>()
                .Join<User,Profile>(nameof(User.Id),nameof(Profile.UserId))
                .QuerySelectAsync(conn);
                
    }

    List<MyNewObject> myNewObjects =
        queryBuilder.MapTo<User, Profile,MyNewObject>(
            (u, p) => new MyNewObject()
            {
                Id = u.Id,
                Fname = u.FirstName,
                Lname = u.LastName,
                AvName = p.AvatarName
            }).ToList();
```

> You can map up to 7 tables into a single object.

[^ Back To Top][TableOfContents]

#### Order By

> You can call `orderBy("PropertyName",OrderByTypes)` Method as many times you need.
> 2end parameter is an ENUM with values of `Asc,Desc` and default value is `Asc`.
> Below example shows the usage of OrderBy method both ascending and descending:

```csharp
    var queryBuilder = new EzQueryBuilder();
    await using (var conn = new SqlConnection(Settings.ConnectionString))
    {
        await queryBuilder.From<User>()
                .From<Profile>()
                .Join<User,Profile>(nameof(User.Id),nameof(Profile.UserId))
                .OrderBy<User>(nameof(User.LastName))
                .OrderBy<Profile>(nameof(Profile.AvatarName),OrderByTypes.Desc)
                .QuerySelectAsync(conn);
                
    }
```

[^ Back To Top][TableOfContents]

#### Where

> Using lambda expression, you can pass a predicate to `.Where(predicate)` function.
> keep in mind that, we are supporting simple operations and for complex queries you need
> to use the [ORM functionality][UseAsOrm] of EzQuery.

> **Warning:** You can not use the *Where* function multiple times in a single query.
> If you do so,
> the last *Where* function will be used in your query.

>However, you can declare conditions to up to 7 tables in your Where function if you
>want. if you use any not supported operation in the predicate, you will get an exception
>indicating the problem. you wont get any silent treatment so feel free to experiment
>and inform us what went wrong and how we can improve this library.
>Below example shows the above join query with **Where** function:

```csharp
    var queryBuilder = new EzQueryBuilder();
    await using (var conn = new SqlConnection(Settings.ConnectionString))
    {
        await queryBuilder.From<User>()
                .From<Profile>()
                .Join<User,Profile>(nameof(User.Id),nameof(Profile.UserId))
                //
                .Where<User,Profile>( (u,p)=> u.FirstName.StartsWith("Test") && p.AvatarName.Contains("ed") && u.Id > 254)
                //
                .OrderBy<User>(nameof(User.LastName))
                .OrderBy<Profile>(nameof(Profile.AvatarName),OrderByTypes.Desc)
                .QuerySelectAsync(conn);
                
    }
```

[^ Back To Top][TableOfContents]

#### Fetch Skip

> For fetching and skipping rows, you can use `.Limit(take:10,skip:5)` function.
> If you don't want to skip or don't want to limit the number of returned records,
> you can set the corresponding parameter to `0`.

```csharp
     //Take 10 , Skip 5
    .Limit(10,5)
     //Take 10 , Do not skip
    .Limit(10)
     //Skip 5 return the rest
    .Limit(0,5)
    //Skip 5 return the rest
    .Limit(skip:5)
```

[^ Back To Top][TableOfContents]

#### Paged Results

> Using ezQuery makes paging the results so simple. If you use the `.Paginate(currentPage:1,itemsPerPage:20)`
> function, the `.Limit(take:10,skip:5)` function will be ignored. You will get the limited number of results
> using `.GetResult<T>()` function and overall number of records can be obtained by `.GetPagedTotalRowsCount()` function.
> Here is an example:

```csharp
    var queryBuilder = new EzQueryBuilder();
    await using (var conn = new SqlConnection(Settings.ConnectionString))
    {
        await queryBuilder.From<User>()
                .From<Profile>()
                .Join<User,Profile>(nameof(User.Id),nameof(Profile.UserId))
                .Where<User,Profile>( (u,p)=> u.Name.StartsWith("Test") && p.AvatarName.Contains("ed") && u.Id > 254)
                .OrderBy<User>(nameof(User.LastName))
                .OrderBy<Profile>(nameof(Profile.AvatarName),OrderByTypes.Asc)
                .Paginate(currentPage:1,itemsPerPage:20)
                .QuerySelectAsync(conn);
                
    }

    List<MyNewObject> myNewObjects =
        queryBuilder.MapTo<User, Profile,MyNewObject>(
            (u, p) => new MyNewObject()
            {
                Id = u.Id,
                Fname = u.Name,
                Lname = u.LastName,
                AvName = p.AvatarName
            }).ToList();

    int TotalCount = queryBuilder.GetPagedTotalRowsCount();
```

[^ Back To Top][TableOfContents]

#### Update

> Updating a table is as simple as using the `Set<T>()` function,
> but you should consider using  `Where<T>()` function as well 
> unless you are going to update the entire table.
> 
>If you want to
> set some columns to *NULL*, you can.
> 
>The `Set<T>()` function has 4 overloads. Let's look at them one by one:

> **Example 1 :** Set `FirstName = "Eric"` and set any other property to `NULL`.
> Leave `User.Id` or any property that is decorated with `[key]` attribute as it is.

```csharp
    var queryBuilder = new EzQueryBuilder();
    var user = new User()
    {
        Id = 21,
        FirstName = "Eric"
    };
    queryBuilder.Set<User>(user)
        .Where<User>(p => p.Id == 661533);
    int affectedRows = await queryBuilder.ExecuteUpdateAsync(conn);
```

> **Example 2 :** Set `FirstName = "Eric"` and set `Id = 21` as well
> and set any other property to `NULL`.

```csharp
    var queryBuilder = new EzQueryBuilder();
    var user = new User()
    {
        Id = 21,
        FirstName = "Eric"
    };

    queryBuilder.Set<User>(user,includeKey:true)
        .Where<User>(p => p.Id == 661533);
    int affectedRows = await queryBuilder.ExecuteUpdateAsync(conn);
```

> **Example 3 :** Set `FirstName = "Eric"` and set `Id = 21` as well
> and leave any other property as it is.

```csharp
    var queryBuilder = new EzQueryBuilder();
    // Creating anonymous object with 2 properties: Id and FirstName
    object user = new 
    {
        Id = 21,
        FirstName = "Eric"
    };

    queryBuilder.Set<User>(user,includeKey:true)
        .Where<User>(p => p.Id == 661533);
    int affectedRows = await queryBuilder.ExecuteUpdateAsync(conn);
```

> **Example 4 :** Set properties one by one
> and leave any other property as it is.

```csharp
    var queryBuilder = new EzQueryBuilder();

    queryBuilder.Set<User>(nameof(User.Id),21)
        .Set<User>(nameof(User.FirstName),"Eric")
        .Set<User>(nameof(User.BirthDay),null)
        .Where<User>(p => p.Id == 661533);
    int affectedRows = await queryBuilder.ExecuteUpdateAsync(conn);
```

[^ Back To Top][TableOfContents]

#### Insert

> You can call most of the EzQuery Methods multiple times for a single query.
> `InsertValues<T>(IEnumerable<T> items, includeKey = false)`
> and `InsertValue<T>(T item, includeKey = false)` are some of them.
> `includeKey` is `false` by default but if you set it to `true`, it means you are going to insert the Ids as well as other properties.

> **Example 1 :** insert list of values and get inserted Ids back.

```csharp
        var users = new List<User>(){
            new User(){
                FirstName = "Eric",
                LastName = "Test"
            }
        };
        await using var conn = new SqlConnection(Settings.ConnectionString);
        var queryBuilder = new EzQueryBuilder()
                                .InsertValues(users);           
        var effectedRows = await queryBuilder.ExecuteInsertAsync(conn);
        List<int> insertedIds = queryBuilder.GetInsertedIds();
```

> **Example 2 :** insert a single value and get inserted Id back.

```csharp
        var user =  new User(){
                                FirstName = "Eric",
                                LastName = "Test"
                            };
       
        var queryBuilder = new EzQueryBuilder()
                                .InsertValue(user);           
        var effectedRows = await queryBuilder.ExecuteInsertAsync(conn);
        List<int> insertedIds = queryBuilder.GetInsertedIds();
```

[^ Back To Top][TableOfContents]

#### Delete

> Delete is so simple with EzQuery but keep in mind that you should use `Where<T>()`
> unless you are going to delete every single record in your table.

> **Example 1 :** Delete a user with Id of 21.

```csharp
    var queryBuilder = new EzQueryBuilder();

    int affectedRows =  await queryBuilder
                        .Where<User>(u => u.Id == 21)
                        .ExecuteDeleteAsync<User>(conn);
```

> **Example 2 :** Delete every single record in **Users** Table.

```csharp
    var queryBuilder = new EzQueryBuilder();

    int affectedRows =  await queryBuilder
                        .ExecuteDeleteAsync<User>(conn);
```

[^ Back To Top][TableOfContents]

### Use as an ORM

> For any reason if you can not use query building functionality of this package,
> you can simply just write down your SQL Command and pass parameters as an anonymous object
> to EzQuery and get your result back populated with the given object.


[^ Back To Top][TableOfContents]

#### Query Database

> below object is for the sake of this demonstration and as you see,
> it is not even inherited form `:EzQueryTable<MyNewObject>`

```csharp
    public class MyNewObject
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string LastName { get; set; }
        public string BirthDay { get; set; }
        public decimal Credit { get; set; }
    }
```

> **Example 1 :** Select from `Users` Table into `MyNewObject` object

```csharp
    await using var conn = new SqlConnection(Settings.ConnectionString);
    var queryBuilder = new EzQueryBuilder();
    List<MyNewObject> myUsers = await queryBuilder
                                .QueryAsync<MyNewObject>(conn, "SELECT * FROM Users WHERE Id=@Id",new {Id = 21});
```

[^ Back To Top][TableOfContents]

#### Execute commands on Database

> **Example 2 :** Update `LastName` from `Users` Table where `Id=21`;

```csharp
    await using var conn = new SqlConnection(Settings.ConnectionString);
    var queryBuilder = new EzQueryBuilder();
    int affectedRows = await queryBuilder
                                .ExecuteAsync(conn, "UPDATE Users Set LastName=@PreferredName WHERE Id=@Id",
                                new {Id = ,PreferredName="TestLastName"});
```

[^ Back To Top][TableOfContents]

### Use repository pattern helper methods

> Probably you asked yourself ,what is the repository pattern?
> 
> Well, as [Joydip Kanjilal](https://www.infoworld.com/author/Joydip-Kanjilal/) says,
> "The Repository pattern implements separation of concerns by abstracting the data persistence logic in your applications". His full article is available [here](https://www.infoworld.com/article/3107186/how-to-implement-the-repository-design-pattern-in-c.html#:~:text=The%20Repository%20pattern%20implements%20separation,persistence%20logic%20in%20your%20applications&text=In%20using%20the%20Repository%20design,%2C%20an%20xml%20file%2C%20etc.).

> In other words,by using the repository pattern, you have a single source of truth
> and by any reason you need to make critical changes in your data persistence logic
> you just need to do it in one place and not all over your code.

[^ Back To Top][TableOfContents]

#### Recommended folder structure

> Now that we know what [the repository pattern][UseAsRepository] is and how to create a [table model][TableModel]
> , here is how we layout files and folders for `Users` and `Profiles` tables.
> Don't worry, it is not as complex as it seems to be. In fact once you learn
> the pattern you can create the whole database model 10 times faster and more efficiently.

```text
.
+-- Repo
    +-- Models
    |   +-- Profile.cs
    |   +-- User.cs
    +-- Interfaces
    |   +-- IProfiles.cs
    |   +-- IUsers.cs
    +-- Repositories
    |   +-- Profiles.cs
    |   +-- Users.cs
    +-- DataContext.cs
```

[^ Back To Top][TableOfContents]

#### Recommended DataContext class

> In some cases developers prefer to get the connection string directly from the config file.

```csharp
using System;
using EzQueryTest.Repo.Repositories;
using Microsoft.Data.SqlClient;

namespace EzQueryTest.Repo
{
    public interface IDataContext
    {
        Users Users { get; }
        Profiles Profiles { get; }
    }

    public class DataContext : IDisposable, IDataContext
    {
        private readonly string _connectionString;
        private readonly SqlConnection _context;

        public Users Users { get; }
        public Profiles Profiles { get; }

        public DataContext(string connectionString)
        {
            _connectionString = connectionString;
            _context = new SqlConnection(_connectionString);
            Users = new Users(_context);
            Profiles = new Profiles(_context);
        }


        public void Dispose()
        {
            _context?.Dispose();
        }
    }
}
```

[^ Back To Top][TableOfContents]

#### Recommended repository class and it's interface

> Just by inheriting from `EzQueryHelperClass<T>` in `ZeidLab.EzQuery.Helpers` namespace,
> you inherit [30 commonly used functions and overloads][RepositoryMethods].
> If you need any costume function, you can easily define it in your class
> and use it side by side with already defined helper methods.

> Let's look at **IUsers.cs** and **IProfiles.cs** file

```csharp
    using EzQueryTest.Repo.Models;
    using ZeidLab.EzQuery.Helpers;

    namespace EzQueryTest.Repo.Interfaces
    {
        public interface IUsers : IEzQueryHelperClass<User>
        {
            
        }
    }
```

```csharp
    using EzQueryTest.Repo.Models;
    using ZeidLab.EzQuery.Helpers;

    namespace EzQueryTest.Repo.Interfaces
    {

        public interface IProfiles : IEzQueryHelperClass<Profile>

        {

        }
    }
```

> Let's look at **Users.cs** and **Profiles.cs** file

```csharp
    using EzQueryTest.Repo.Interfaces;
    using EzQueryTest.Repo.Models;
    using Microsoft.Data.SqlClient;
    using ZeidLab.EzQuery.Helpers;

    namespace EzQueryTest.Repo.Repositories
    {
        public class Users : EzQueryHelperClass<User>, IUsers

        {
            public Users(SqlConnection context) : base(context)
            {
            }
        }
    }
```

```csharp
    using EzQueryTest.Repo.Interfaces;
    using EzQueryTest.Repo.Models;
    using Microsoft.Data.SqlClient;
    using ZeidLab.EzQuery.Helpers;

    namespace EzQueryTest.Repo.Repositories
    {
        public class Profiles : EzQueryHelperClass<Profile>,IProfiles
        {
            public Profiles(SqlConnection context) : base(context)
            {
            }
        }
    }
```

[^ Back To Top][TableOfContents]

#### EzQuery Repository Pattern Helper Methods

> First of all, EzQuery Helper methods, a little bit violate the repository pattern
> principles allowing developers to pass lambda expressions as a parameter and using
> them as part of the main query. The problem is, if in any case you want to use
> another library other than EzQuery after finishing your application, probably
> you will need to go back and refactor many places in your app. I strongly
> recommend you to define your own helper methods based on your own needs and use
> EzQuery helper methods as examples.

> **SELECT Examples :**

```csharp
    using (var Db = new DataContext(Settings.ConnectionString))
    {
        var users = await Db.Users.SelectAsync());
    }
```

```csharp
    using (var Db = new DataContext(Settings.ConnectionString))
    {
        int id = 20;
        var user = (await Db.Users.SelectAsync(p => p.Id == id)).FirstOrDefault();
    }
```

```csharp
    using (var Db = new DataContext(Settings.ConnectionString))
    {
        PagedQueryResult<User> result = await Db.Users.SelectPagedAsync();
    }
```

```csharp
    using (var Db = new DataContext(Settings.ConnectionString))
    {
        PagedQueryResult<User> result = 
            await Db.Users.SelectPagedAsync(
                            predicate: p => p.LastName.Contains("ed"),
                            currentPage:3,
                            itemsPerPage:20);

        List<User> users = result.Records;
        int totalNumberOfRecordsInDb = result.TotalCount;
    }
```

[^ Back To Top][TableOfContents]

> **COUNT Examples :**

```csharp
    using (var Db = new DataContext(Settings.ConnectionString))
    {
        int count = await Db.Users.CountAsync();
    }
```

```csharp
    using (var Db = new DataContext(Settings.ConnectionString))
    {
        int count = await Db.Users
            .CountAsync(p => p.BirthDay >= DateTime.UtcNow.AddYears(-18)
            || p.BirthDay == null);
    }
```

[^ Back To Top][TableOfContents]

> **UPDATE Examples :**

```csharp
    using (var Db = new DataContext(Settings.ConnectionString))
    {
        int affectedRows = await Db.Users.UpdateAsync(u => u.Id == 19,
        new { LastName = "Test Update" });
    }
```

```csharp
    using (var Db = new DataContext(Settings.ConnectionString))
    {
        User user = (await Db.Users.SelectAsync(p => p.Id == 20))
            .FirstOrDefault();
        user.LastName = "Update Test";
        user.Id = 358;

        int affectedRows = await Db.Users
            .UpdateAsync(u => u.Id == 20, user, includeKey:true);
    }
```

```csharp
    using (var Db = new DataContext(Settings.ConnectionString))
    {
        int affectedRows = await Db.Users
            .UpdateAsync(u => u.Id == 19, nameof(User.LastName), "Update Test");
    }
```

[^ Back To Top][TableOfContents]

> **INSERT Examples :**

```csharp
    var users = new List<User>();
    users.Add(new User());
    users.Add(new User());
    users.Add(new User());

    using (var Db = new DataContext(Settings.ConnectionString))
    {
        int affectedRows = await Db.Users.InsertAsync(users);
    }

    using (var Db = new DataContext(Settings.ConnectionString))
    {
        var LastInsertedIds = (List<int>)await Db.Users.InsertGetIdsAsync(users);
    }

    using (var Db = new DataContext(Settings.ConnectionString))
    {
        int affectedRows = await Db.Users
            .InsertAsync(new User(),includeKeys:true);
    }

    using (var Db = new DataContext(Settings.ConnectionString))
    {
        int LastInsertedId = await Db.Users.InsertGetIdAsync(new User());
    }
```

[^ Back To Top][TableOfContents]

> **DELETE Examples :**

```csharp
    using (var Db = new DataContext(Settings.ConnectionString))
    {
        // DELETE All RECORDS
        int affectedRows = await Db.Users.DeleteAsync();
    }

    using (var Db = new DataContext(Settings.ConnectionString))
    {
        int affectedRows = await Db.Users.DeleteAsync(u => u.Id == 20);
    }
```

[^ Back To Top][TableOfContents]

## What is a table model?

>In order to use this library you need to define a table model.
>Below is a model example of **Users** Table containing **Name,LastName,BirthDay,Credit** and a not mapped properties named **Status** :

```csharp
    [Table("Users")]
    public class User :EzQueryTable<User>
    {
        [Key]
        public int Id { get; set; }
        [Column("Name"),Required]
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public DateTime? BirthDay { get; set; }
        public decimal Credit { get; set; }
        [NotMapped]
        public bool Status { get; set; }
    }
```

> All **Table Models** must inherit from `EzQueryTable<T>`.

> If column is a null-able type and you are going to set a null value to it
> , you should use a null-able type in your model.
> In the above example, **BirthDay** is a null-able type and you can pass NULL to it
> using insert or update command. For more information's you should take a look at
> [Supported Component Model Data Annotations][SupportedAnotations].

> For the sake of this tutorial, we are going to define another table model.
> This time for the **Profiles** table:

```csharp
    [Table("Profiles")]
    public class Profile : EzQueryTable<Profile>
    {
        public int Id { get; set; }
        public int UserId { get; set; }
        public string AvatarName { get; set; }
        [NotMapped] 
        public bool Status { get; set; }
    }
```

[^ Back To Top][TableOfContents]

## Supported Component Model Data Annotations

### What are the namespaces?

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;
```

[^ Back To Top][TableOfContents]

### What are the supported annotations?

>- [Table("TableName")](#table)
>- [Key](#key)
>- [Column("ColumnName")](#column)
>- [NotMapped](#notmapped)
>- [Required](#required)

#### Table

> The only mandatory annotation is `[Table("TableName")]`.
> Without this annotation you can only use the [ORM feature][UseAsOrm] of this package.

#### Key

> In the above [table model][TableModel] example, Using or not using the `[key]` attribute won't affect the result.
> Column name `Id` is automatically considered as `[key]`. But if you choose any other name
> as your **Id** column, you should consider using the `[key]` attribute on it.

#### Column

> `[Column("ColumnName")]` attribute allows you to pick a name other than the column name of
> your database table for your property of your **Table Model**.
> EzQuery only uses the **Name** property of `[Column]` attribute and
> the **Order** and the **TypeName** is ignored.

#### NotMapped

> If you are using a property in your **Table Model** that does not have a
> corresponding column in your database , you should consider using the `[NotMapped]` attribute.

#### Required

> If you have a column in your database that is **NotNull** you should consider using the `[Required]` attribute.

[^ Back To Top][TableOfContents]
