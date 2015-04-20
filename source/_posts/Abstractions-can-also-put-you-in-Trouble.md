title: "Abstractions can also put you in Trouble"
date: 2011-11-19 15:25:05
tags:
---

We all have enjoyed the beauty of abstracting out functionality, so that it simplifies the underlying complexity. Sometimes it can come back and hit you on the forehead like a boomerang if we don’t know what is going underneath those hidden layers.

Recently it happened to me with the LINQ provider. If you have a data source then you can build your own Query provider on top of it, so that the users can access your data using the LINQ statements without understand how it is fetched.

For e.g. [Linq2Twitter] it provides the IQueryable interface which you can use fetch the twitter API without understanding the REST API’s exposed by the Twitter.

Similarly Ayende has created a Linq provider for NHibernate ([NHibernate.Linq][1]) which hides complexities of writing Criteria API’s.

This is what happened to me – we had a repository method which is using the NHibernate.Linq and it returned an IQueryable interface of domain model.

Like to the below code(only an example, this is not from the actual codebase itself)

```cs
public IQueryable<Customer> QueryableCustomers()
{
    session = SessionFactory.OpenSession();
    return session.Linq<Customer>().AsQueryable();
}
```

In the consumer part (Action method in the controller), we are filtering the customer with a name containing particular string using the Contains method

`this.dataProvider.QueryableCustomers().Where(x => x.Name.Contains("custo")).ToList();`

Everything worked as expected, but one day the filtering stopped working, I couldn’t figure it out initially, but when I looked into the `QueryableCustomers()` method, it has been changed to something like this

```cs
public IQueryable<Customer> QueryableCustomers()
{
    session = SessionFactory.OpenSession();
    return session.Linq<Customer>().ToList().AsQueryable();
}
```

Not much difference in the code wise, but internally the earlier code will filter the customers using the SQL statement by adding a where clause to the SQL it generated and later one will fetch all the customers from the DB and filter that result using the LINQ on objects.

SQL generated for initial code

```sql
SELECT
    this_.Id as Id0_0_,
    this_.Name as Name0_0_,
    this_.Address as Address0_0_
FROM
    Customer this_
WHERE
    this_.Name like @p0;
@p0 = '%custo%'
```

After code change, the query generated SQL was

```sql
SELECT
this_.Id as Id0_0_,
this_.Name as Name0_0_,
this_.Address as Address0_0_
  FROM
Customer this_
```

By now you might have realized the reason why the I didn’t get the expected result from the new code change – if not here is the explanation, **Contains** in the earlier code is translated by the NH LINQ provider to a where clause which is case insensitive by default unless you have set any specific Collation in the database or on the column itself but in the other hand new code returned a collection of Customer itself, then we applied String.Contains on the Name property which is case sensitive.

My point here is - we can’t blindly believe in the abstractions provided by anyone without understanding what is happening underneath.

[Linq2Twitter]: http://linqtotwitter.codeplex.com/
[1]: https://nuget.org/List/Packages/NHibernate.Linq
