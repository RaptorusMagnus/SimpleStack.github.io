---
title: SimpleStack - Querying Data - Typed
hero_title:
layout: page
hero_image: /assets/background.jpg
show_sidebar: false
menubar: home_menu
---
# Typed select statements
All these samples are based on the"Dog" class from the [home page](/) and a connection as described in the [setup](/setup/sqlserver):

## WHERE clause generation using strongly typed LINQ queries

<div class="notification is-info">
<span class="icon">
  <i class="fas fa-info-circle"></i>
</span>
Queries output are given for information only. 
<ul>
<li>They may change from one database to another</li>
<li>All generated queries will use parameters instead of string hardcoded values</li>
</ul>
</div>

### Equals, Not equals, Bigger than, Less than, Like, is Null...

```csharp
db.SelectAsync<Dog>(q => q.Name == "Rex");       // WHERE ("Name" = 'Rex')
db.SelectAsync<Dog>(q => q.Name != "Rex");       // WHERE ("Name" <> 'Rex')
db.SelectAsync<Dog>(q => q.Weight == 10);        // WHERE ("Weight" = 10)
db.SelectAsync<Dog>(q => q.Weight > 10);         // WHERE ("Weight" > 10)
db.SelectAsync<Dog>(q => q.Weight >= 10);        // WHERE ("Weight" >= 10)
db.SelectAsync<Dog>(q => q.Weight < 10);         // WHERE ("Weight" < 10)
db.SelectAsync<Dog>(q => q.Weight <= 10);        // WHERE ("Weight" <= 10)
db.SelectAsync<Dog>(q => q.Name.Contains("R"));  // WHERE ("Name" LIKE("%R%"))
db.SelectAsync<Dog>(q => q.Name.StartWith("R")); // WHERE ("Name" LIKE("R%"))
db.SelectAsync<Dog>(q => q.Name.EndWidth("R"));  // WHERE ("Name" LIKE("%R"))
db.SelectAsync<Dog>(q => q.Name != null);        // WHERE ("Name" IS NOT NULL)
db.SelectAsync<Dog>(q => q.BirthDate.HasValue);  // WHERE ("BirthDate" IS NOT NULL)
db.SelectAsync<Dog>(q => !q.BirthDate.HasValue); // WHERE ("BirthDate" IS NULL)
db.SelectAsync<Dog>(q => q.BirthDate.Month = 10) // WHERE (MONTH("BirthDate") = 10)
```

### AND or OR

```csharp
// WHERE ("Name" LIKE 'R' OR "Weight" > 10)
db.Select<Dog>(q => q.Name.Contains("R") || q.Weight > 10); 

// WHERE ("Name" LIKE 'R' AND "Weight" > 10)
db.Select<Dog>(q => q.Name.Contains("R") && q.Weight > 10); 
```
## Select Distinct
```csharp
// SELECT DISTINCT FROM Dogs
db.Select<Dog>(x => {
    x.Distinct();
}); 
```
## Change from
The Table name by default is derived from the Poco class name (optionally decorated by [Attibutes](../attributes))
```csharp
// SELECT * FROM someotherschema.Someothertable
db.Select<Dog>(x => {
    x.From("Someothertable","someotherschema");
}); 
```
## Select only some fields
```csharp
// SELECT Breed, Name FROM Dogs
db.Select<Dog>(x => {
    x.Select(y => new {y.Name,y.Breed})
}); 
```
## Aggregations
```csharp
// SELECT MAX("BirthDate") FROM DOG
conn.GetScalar<Dog, DateTime>(x => Sql.Max(x.BirthDate));
// SELECT AVG("Weight") FROM DOG
conn.GetScalar<Dog, decimal>(x => Sql.Avg(x.Weight));
// SELECT  FROM DOG 
//    WHERE Name LIKE '%A%'
//    GROUP BY Breed
conn.Select<Dog>(x => {
   x.Select(y => new Dog{Age= Sql.Avg(y.Age), y.Breed}); 
   x.Where(y => y.Name.Contains("A");
   x.GroupBy(y => y.Breed);
```
## Order By
```csharp
// SELECT * FROM Dogs ORDER BY Breed
db.Select<Dog>(x => {
    x.OrderBy(y => y.Breed);
}); 
```
## Limits
```csharp
// SELECT * FROM Dogs LIMIT(20,40)
db.Select<Dog>(x => {
    x.Limit(20,40);
}); 
```
## IN Criteria

```csharp
string[] breeds = new {"Beagle", "Border Collie", "Golden Retriever"};
db.Select<Dog>(q => breeds.Contains(g.Breed));  
// WHERE "Breed" In ('Beagle', 'Border Collie', 'Golden Retriever')
```

## Sql helper class 

### Date part methods
Use the date function (specific for each database)
```csharp
// SELECT YEAR("BirthDate") FROM DOG
conn.GetScalar<Dog, int>(x => Sql.Year(x.BirthDate))
// SELECT "Id","Name","Breed","DareBirth","Weight" FROM DOG WHERE MONTH("BirthDate") = 10
conn.Select<Dog>(x => Sql.Month(x.BirthDate) = 10)
```

### Aggregation function

```csharp
// SELECT MAX("BirthDate") FROM DOG
conn.GetScalar<Dog, DateTime>(x => Sql.Max(x.BirthDate))
// SELECT AVG("Weight") FROM DOG
conn.GetScalar<Dog, decimal>(x => Sql.Avg(x.Weight))
```

# Method signatures
```csharp

Task<IEnumerable<T>> SelectAsync<T>(CommandFlags flags = CommandFlags.Buffered);
Task<IEnumerable<T>> SelectAsync<T>(Expression<Func<T, bool>> predicate, 
                                    CommandFlags flags = CommandFlags.Buffered);
Task<IEnumerable<T>> SelectAsync<T>(Action<TypedSelectStatement<T>> expression, 
                                    CommandFlags flags = CommandFlags.Buffered);

Task<T> FirstAsync<T>(Expression<Func<T, bool>> predicate);
Task<T> FirstAsync<T>(Action<TypedSelectStatement<T>> expression);

Task<T> FirstOrDefaultAsync<T>(Expression<Func<T, bool>> predicate);
Task<T> FirstOrDefaultAsync<T>(Action<TypedSelectStatement<T>> expression);

Task<TKey> GetScalarAsync<T, TKey>(Expression<Func<T, TKey>> field);
Task<TKey> GetScalarAsync<T, TKey>(Expression<Func<T, TKey>> field, 
                                   Expression<Func<T, bool>> predicate);

Task<long> CountAsync<T>(Expression<Func<T, bool>> expression);
Task<long> CountAsync<T>(Action<TypedSelectStatement<T>> expression);
```