# 🍃 NoSQL / MongoDB Quick Reference

## NoSQL Types

| Type           | Examples           | Use Case |
|----------------|--------------------|----------|
| Document store | MongoDB, Couchbase | Semi-structured data, flexible schema |
| Key-Value      | Redis, DynamoDB    | Caching, sessions, leaderboards |
| Wide-Column    | Cassandra, HBase   | Time series, large scale writes |
| Graph          | Neo4j, Amazon Neptune | Relationships, social networks |

## MongoDB Basics

### CRUD Operations

```js
// Insert
db.users.insertOne({ name: "Alice", age: 30 });
db.users.insertMany([{ name: "Bob" }, { name: "Carol" }]);

// Find
db.users.find({});                               // all documents
db.users.find({ age: 30 });                      // filter
db.users.findOne({ name: "Alice" });             // first match
db.users.find({ age: { $gt: 25 } }).limit(10);  // operators + limit

// Update
db.users.updateOne(
    { name: "Alice" },
    { $set: { age: 31 } }
);
db.users.updateMany(
    { age: { $lt: 18 } },
    { $set: { category: "minor" } }
);
db.users.replaceOne({ name: "Alice" }, { name: "Alice", age: 32 });

// Delete
db.users.deleteOne({ name: "Bob" });
db.users.deleteMany({ age: { $lt: 18 } });

// Upsert
db.users.updateOne(
    { email: "alice@example.com" },
    { $set: { name: "Alice" } },
    { upsert: true }
);
```

### Query Operators

```js
// Comparison
{ age: { $gt: 25 } }    // greater than
{ age: { $gte: 25 } }   // >=
{ age: { $lt: 30 } }    // <
{ age: { $lte: 30 } }   // <=
{ age: { $ne: 25 } }    // not equal
{ age: { $in: [25, 30, 35] } }
{ age: { $nin: [25, 30] } }

// Logical
{ $and: [{ age: { $gt: 20 } }, { age: { $lt: 40 } }] }
{ $or:  [{ name: "Alice" }, { name: "Bob" }] }
{ $not: { age: { $gt: 30 } } }
{ $nor: [{ age: 25 }, { name: "Bob" }] }

// Element
{ field: { $exists: true } }
{ field: { $type: "string" } }

// Array
{ tags: { $all: ["js", "python"] } }
{ tags: { $elemMatch: { $gt: 10, $lt: 20 } } }
{ tags: { $size: 3 } }

// Text search (requires text index)
{ $text: { $search: "javascript framework" } }

// Regex
{ name: { $regex: /^alice/i } }
```

### Update Operators

```js
// Field
{ $set:    { field: value } }
{ $unset:  { field: "" } }
{ $rename: { oldName: "newName" } }
{ $inc:    { age: 1 } }     // increment
{ $mul:    { price: 1.1 } } // multiply
{ $min:    { score: 50 } }  // set if current > 50
{ $max:    { score: 100 } } // set if current < 100

// Array
{ $push:  { tags: "new-tag" } }
{ $pull:  { tags: "old-tag" } }
{ $pop:   { tags: 1 } }      // 1 = last, -1 = first
{ $addToSet: { tags: "unique-tag" } }  // only if not exists
{ $push:  { scores: { $each: [1,2,3], $sort: 1 } } }
```

### Projection

```js
// Include fields (1 = include, 0 = exclude)
db.users.find({}, { name: 1, email: 1, _id: 0 });

// Exclude fields
db.users.find({}, { password: 0, __v: 0 });
```

### Sorting, Skip, Limit

```js
db.users
    .find({})
    .sort({ age: -1, name: 1 })  // -1 DESC, 1 ASC
    .skip(20)
    .limit(10);
```

### Aggregation Pipeline

```js
db.orders.aggregate([
    // Stage 1: Filter
    { $match: { status: "completed" } },

    // Stage 2: Join (lookup)
    {
        $lookup: {
            from: "users",
            localField: "user_id",
            foreignField: "_id",
            as: "user"
        }
    },

    // Stage 3: Unwind array
    { $unwind: "$user" },

    // Stage 4: Group
    {
        $group: {
            _id: "$user.country",
            total_orders: { $sum: 1 },
            total_amount: { $sum: "$amount" },
            avg_amount:   { $avg: "$amount" }
        }
    },

    // Stage 5: Sort
    { $sort: { total_amount: -1 } },

    // Stage 6: Limit
    { $limit: 10 },

    // Stage 7: Project (reshape output)
    {
        $project: {
            country: "$_id",
            total_orders: 1,
            total_amount: { $round: ["$total_amount", 2] },
            _id: 0
        }
    }
]);
```

### Indexes

```js
// Create
db.users.createIndex({ email: 1 });                  // single
db.users.createIndex({ name: 1, age: -1 });          // compound
db.users.createIndex({ email: 1 }, { unique: true }); // unique
db.users.createIndex({ bio: "text" });                // text search
db.users.createIndex({ location: "2dsphere" });       // geospatial
db.users.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 }); // TTL

// View
db.users.getIndexes();

// Drop
db.users.dropIndex("email_1");
```

## Redis Quick Reference

```
# String
SET key "value"
GET key
SETEX key 3600 "value"    # with TTL in seconds
INCR counter
INCRBY counter 5

# List (stack / queue)
LPUSH list "a"
RPUSH list "b"
LPOP list
RPOP list
LRANGE list 0 -1          # all elements

# Set
SADD myset "a" "b"
SMEMBERS myset
SISMEMBER myset "a"
SUNION set1 set2
SINTER set1 set2

# Hash (object)
HSET user name "Alice" age 30
HGET user name
HGETALL user
HDEL user age

# Sorted Set (leaderboard)
ZADD scores 100 "Alice"
ZADD scores 200 "Bob"
ZRANGE  scores 0 -1 WITHSCORES  # asc
ZREVRANGE scores 0 2 WITHSCORES  # top 3

# Expiry
EXPIRE key 60         # TTL in seconds
TTL key               # remaining TTL
PERSIST key           # remove TTL
```

## When to Use SQL vs NoSQL

| Criteria | SQL | NoSQL |
|----------|-----|-------|
| Schema | Fixed schema | Flexible/schema-less |
| Relationships | Complex joins | De-normalized, embedded |
| Transactions | ACID | BASE (eventual consistency) |
| Scalability | Vertical (mostly) | Horizontal |
| Query complexity | Complex queries | Simple lookups |
| Use cases | Finance, ERP, CRM | Content, catalog, social, IoT |
