# EzQuery

## What is EzQuery?
 > __EzQuery__  is a  super easy to use  yet powerful .Net Core 5 library,
 to build SQL queries, to execute them , and to return populated sets of results of given
 table models. It does not contain a lot of features but it does the job.


# Table of Contents

[TableOfContents]:#table-of-Contents
[TableModel]:#what-is-a-table-model
[SupportedAnotations]:#what-are-the-supported-annotations
[UseAsBuilder]:#use-as-a-query-builder
[UseAsOrm]:#use-as-an-orm
[UseAsRepository]:#use-repository-pattern-helper-methods

- [How to install?](#how-to-install)
- [How to use?](#how-to-use)
  - [Use as a query builder][UseAsBuilder]
    - [Select](#select)
    - [Join](#join)
    - [Map to](#map-to)
    - [Order By](#order-by)
    - [Where](#where)
    - [Fetch Skip](#fetch-skip)
    - [Paged Results](#paged-results)
    - [Update](#update)
    - [Insert](#insert)
    - [Delete](#delete)
  - [Use as an ORM][UseAsOrm]
    - [Query Database](#query-database)
    - [Execute commands on Database](#execute-commands-on-database)
  - [Use repository pattern helper methods][UseAsRepository]
    - [Recommended folder structure](#recommended-folder-structure)
    - [Recommended DataContext class](#recommended-datacontext-class)
    - [Recommended repository class and it's interface](#recommended-repository-class-and-its-interface)
- [What is a table model?][TableModel]
- [Supported Component Model Data Annotations](#supported-component-model-data-annotations)
  - [What are the namespaces?](#what-are-the-namespaces)
  - [What are the supported annotations?][SupportedAnotations]
    - [Table](#table)
    - [Key](#key)
    - [Column](#column)
    - [NotMapped](#notmapped)
    - [Required](#required)
## How to install?
> Using **Nuget PackageManager**, you need to look for **ZeidLab.EzQuery**

> Using **Command line** : `Install-Package ZeidLab.EzQuery -ProjectName MyProject`

[^ Back To Top][TableOfContents]
## How to use?
> There is 3 ways to use this Nuget Package , [As a Query Builder and ORM][UseAsBuilder] 
> , [Just as an ORM][UseAsOrm] or [As a repository pattern][UseAsRepository].
> Of course you have the ability to choose any combination you need as well.


[^ Back To Top][TableOfContents]
### Use as a query builder
> To use query building feature of EzQuery which is the most important and the main reason of building this package 
> , you need to define a [Table Model][TableModel] first. Using ` Microsoft.Data.SqlClient`
> , create an instance of `SqlConnection ` and using below examples create your own SQL query.

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

> You can get value of specific properties by passing their name to `From<T>()`
>  function like below example.In this example only Id and FirstName will be populated
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
> The 3rd parrameter is an ENUM and you can use it like below:

```csharp
    .Join<User,Profile>(nameof(User.Id),nameof(Profile.UserId),JoinTypes.INNER)

    .Join<User,Profile>(nameof(User.Id),nameof(Profile.UserId),JoinTypes.FULL)

    .Join<User,Profile>(nameof(User.Id),nameof(Profile.UserId),JoinTypes.LEFT)

    .Join<User,Profile>(nameof(User.Id),nameof(Profile.UserId),JoinTypes.RIGHT)
```

[^ Back To Top][TableOfContents]
#### Map To

> In above Join Example lets sopuse we need only **Id,FirstName,LastName** from **Users**
>  table and only **AvatarName** from **Profiles** table, and we want to get it as a single object.
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

> Using lamda expression, you can pass a predicate to `.Where(predicate)` function.
> keep in mind that, we are supporting simple operations and for complex queries you need 
> to use the [ORM functionality][UseAsOrm] of EzQuery.

> **Warning:** You can not use the *Where* function multiple times in a single query.
>  If you do so,
> the last *Where* function will be used in your query.

>However, you can declare conditions to up to 7 tables in your Where function if you 
>want. if you use any not supported operation in the predicate, you will get an exception 
>indicating the problem. you wont get any silent treatment so feel free to exprement
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

> For fetching and skiping rows, you can use `.Limit(take:10,skip:5)` function.
> If you dont want to skip or dont want to limit the number of returned recoreds,
> you can set the coresponding parameter to `0`.

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
> using `.GetResult<T>()` function and overall number of recoreds can be obtained by `.GetPagedTotalRowsCount()` function.
> Here is an excample:

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

> Updating a table is as simle as useing the `Set<T>()` function,
> but you should consider using  `Where<T>()` function as well 
> unless you are going to update the entire table.
> 
>If you want to
> set some columns to *NULL*, you can.
> 
>The `Set<T>()` function has 4 overloads. Lets look at them one by one:

> **Example 1 :** Set `Firstname = "Eric"` and set any other property to `NULL`.
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

> **Example 2 :** Set `Firstname = "Eric"` and set `Id = 21` as well
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

> **Example 3 :** Set `Firstname = "Eric"` and set `Id = 21` as well
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

> Lorem Ipsom

[^ Back To Top][TableOfContents]
#### Delete

> Delete is so simple with EzQuery but keep in mind that you should use `Where<T>()` 
> unless you are going to delete every single recored in your table.

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

> Lorem Ipsom

[^ Back To Top][TableOfContents]
#### Query Database

> Lorem Ipsom

[^ Back To Top][TableOfContents]
#### Execute commands on Database

> Lorem Ipsom

[^ Back To Top][TableOfContents]
### Use repository pattern helper methods

> Lorem Ipsom

[^ Back To Top][TableOfContents]
#### Recommended folder structure

> Lorem Ipsom

[^ Back To Top][TableOfContents]
#### Recommended DataContext class

> Lorem Ipsom

[^ Back To Top][TableOfContents]
#### Recommended repository class and it's interface


> Lorem Ipsom

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

> If column is a nullable type and you are going to set null value to it
> , you should use a nullable type in your model.
> In above example, **BirthDay** is a nullable type and you can pass NULL to it
> using insert or update command. For more informations you should take a look at 
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

> `[Column("CoulmnName")]` attribute allows you to pick a name other than the column name of
>  your database table for your property of your **Table Model**.
> EzQuery only uses the **Name** property of `[Column]` attribute and
> the **Order** and the **TypeName** is ignored.

#### NotMapped

> If you are using a property in your **Table Model** that does not have a 
> corresponding column in your database , you should consider using the `[NotMapped]` attribute.

#### Required

> If you have a column in your database that is **NotNull** you should consider using the `[Required]` attribute.

[^ Back To Top][TableOfContents]

