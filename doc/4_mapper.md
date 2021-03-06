# Documentation
## Working with user-defined case classes and tuples

This section describes how to store Cassandra rows in Scala tuples or objects of your own classes.

### Mapping rows to tuples
Instead of mapping your Cassandra rows to objects of the `CassandraRow` class, you can directly 
unwrap column values into tuples of desired type.
 
```scala
sc.cassandraTable[(String, Int)]("test", "words").select("word", "count").toArray
// Array((bar,20), (foo,10))

sc.cassandraTable[(Int, String)]("test", "words").select("count", "word").toArray
// Array((20,bar), (10,foo))
```    

### Mapping rows to (case) objects
Define a case class with properties named the same as the Cassandra columns. 
For multi-word column identifiers, separate each word by an underscore in Cassandra, 
and use the camel case convention on the Scala side. Then provide the explicit class name
when invoking `cassandraTable`:

```scala
case class WordCount(word: String, count: Int)
sc.cassandraTable[WordCount]("test", "words").toArray
// Array(WordCount(bar,20), WordCount(foo,10))
```

The column-property naming convention is:

Cassandra column name	| Scala property name
------------------------|---------------------
`count`	                | `count`
`column_1`	            | `column1`
`user_name`	            | `userName`

Using the same property names as columns also works:

Cassandra column name	| Scala property name
------------------------|---------------------
`COUNT`                 | `COUNT`
`column_1`	            | `column_1`
`user_name`	            | `user_name`

The class doesn't necessarily need to be a case class. The only requirements are:

  - it must be `Serializable`
  - it must have a constructor with parameter names and types matching the columns
  - it must be compiled with debug information, so it is possible to read parameter names at runtime

Property values might be also set by Scala-style setters. The following class is also compatible:
    
```scala
class WordCount extends Serializable {
  var word: String = ""
  var count: Int = 0    
}
```       

### Mapping rows to pairs of objects
You can also map rows to pairs of objects or tuples so that it resemble a mapping key to values.
It is convenient to represent data from Cassandra as an RDD of pairs where the first component is
the primary key and the second one includes all the remaining columns.

Suppose we have a table with the following schema:

```sql
CREATE TABLE test.users (
  user_name     TEXT,
  domain        TEXT,
  password_hash TEXT,
  last_visit    TIMESTAMP,
  PRIMARY KEY (domain, user_name)
);

INSERT INTO test.users (user_name, domain, password_hash, last_visit) VALUES ('john', 'datastax.com', '1234', '2014-06-05');
```

We can access map the rows of this table into pair in the following ways:

```scala
case class UserId(userName: String, domain: String)
case class UserData(passwordHash: String, lastVisit: DateTime)

sc.cassandraTable[KV[UserId, UserData]]("test", "users")

sc.cassandraTable[KV[(String, String), UserData]]("test", "users")

sc.cassandraTable[KV[(String, String), (String, DateTime)]]("test", "users")
```

[Next - Saving data](5_saving.md)
