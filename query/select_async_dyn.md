---
title: SimpleStack - Querying Data - Dynamic
hero_title:
layout: page
hero_image: /assets/background.jpg
show_sidebar: false
menubar: home_menu
---
# Dynamic Select Statements
It is also possible to query tables without having a corresponding Type by specifying the table, schema and column name explicitly.

The return value for dynamic methods is ```IEnumerable<dynamic>```

<div class="notification is-warning">
<span class="icon"><i class="fas fa-exclamation-circle"></i></span>
Dynamic Select Statements are still experimental and API may change in future version of SimpleStack. Feedback is more than welcome !
</div>

```csharp
conn.Select("TestType2",x => x.Select("id","textcol")
                              .Where("textcol",(string y) => y.Substring(0, 1) == "a")
                              .And("id", (int y) => y > 10)
                              .Limit(1, 1));
//SELECT id, textcol FROM TestType2 WHERE SUBSTR(textcol,0,1) = 'a' AND id > 10 LIMIT 1 OFFSET 1
```
The column name or where expression can actually contains any valid SQL, it is therefore possible to do some more complex queries.

```csharp
conn.Select("TestType2",
            x => x.Select("id","CASE WHEN mod(id,2) = 0 THEN 'E' ELSE 'O' END AS even_odd")
                  .Where<char>("CASE WHEN mod(id,2) = 0 THEN 'E' ELSE 'O' END", y => y == 'O'));
// SELECT id FROM TestType2 WHERE (case when mod(id,2) = 0 then 'E' else 'O' end) = 'O'
```

<div class="notification is-warning">
<span class="icon"><i class="fas fa-exclamation-circle"></i></span>
The statements are sent as is to the backend database, therefore pay attention that :
<ul>
<li>It's the caller responsibility to sanitize these statements to avoid SQL injections, ...</li>
<li>The query can contain database specific statements that could break portability to multiple database</li>
</ul>
</div>

You can use the DialectProvider on the Connection to help you generate database specific queries:

```csharp
var idCol = conn.DialectProvider.GetQuotedColumnName("id");
var textCol = conn.DialectProvider.GetQuotedColumnName("textcol")

conn.Select("TestType2",x => x.Select(idCol,textCol)
                              .Where(textCol,
                                     (string y) => y.Substring(0, 1) == "a")
                              .And(idCol, (int y) => y > 10)
                              .Limit(1, 1));
//SELECT id, textcol FROM TestType2 WHERE SUBSTR(textcol,0,1) = 'a' AND id > 10 LIMIT 1 OFFSET 1
```

# Method signatures

Dynamic queries can be used when you do not have a Poco and want to generate database specific queries. 

```csharp
Task<IEnumerable<dynamic>> SelectAsync(string tableName, 
                                       Action<DynamicSelectStatement> selectStatement,
                                       CommandFlags flags = CommandFlags.Buffered)
Task<IEnumerable<dynamic>> SelectAsync(string tableName, 
                                       string schemaName,
                                       Action<DynamicSelectStatement> selectStatement,
                                       CommandFlags flags = CommandFlags.Buffered)
Task<long> CountAsync(string tableName, 
                      Action<DynamicSelectStatement> expression)
Task<long> CountAsync(string tableName,
                      string schemaName, 
                      Action<DynamicSelectStatement> expression)
```
