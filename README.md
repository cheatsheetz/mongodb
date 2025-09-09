# MongoDB Cheat Sheet

Comprehensive reference for MongoDB document database - covering documents, queries, aggregation, indexing, and administration for production environments.

---

## Table of Contents
- [Installation & Connection](#installation--connection)
- [Database & Collection Operations](#database--collection-operations)
- [Document Operations (CRUD)](#document-operations-crud)
- [Query Operations](#query-operations)
- [Aggregation Framework](#aggregation-framework)
- [Indexing](#indexing)
- [Data Modeling](#data-modeling)
- [Replication](#replication)
- [Sharding](#sharding)
- [Administration](#administration)
- [Performance Optimization](#performance-optimization)
- [Security](#security)
- [GridFS](#gridfs)
- [Integration Patterns](#integration-patterns)

---

## Installation & Connection

### Installation
```bash
# Ubuntu/Debian
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo apt update && sudo apt install -y mongodb-org

# macOS (Homebrew)
brew tap mongodb/brew
brew install mongodb-community

# Docker
docker run --name mongodb -d -p 27017:27017 mongo:7.0

# Docker with authentication
docker run --name mongodb -d -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=password mongo:7.0
```

### Connection
```bash
# MongoDB Shell
mongosh

# Connect to specific database
mongosh "mongodb://localhost:27017/myapp"

# Connect with authentication
mongosh "mongodb://username:password@localhost:27017/myapp"

# Connect to replica set
mongosh "mongodb://host1:27017,host2:27017,host3:27017/myapp?replicaSet=rs0"

# Connect with SSL
mongosh "mongodb://localhost:27017/myapp" --tls --tlsCertificateKeyFile client.pem
```

### Connection String Parameters
| Parameter | Description | Example |
|-----------|-------------|---------|
| `authSource` | Authentication database | `mongodb://user:pass@host/db?authSource=admin` |
| `replicaSet` | Replica set name | `mongodb://host1,host2/db?replicaSet=rs0` |
| `readPreference` | Read preference | `mongodb://host/db?readPreference=secondary` |
| `w` | Write concern | `mongodb://host/db?w=majority` |
| `maxPoolSize` | Connection pool size | `mongodb://host/db?maxPoolSize=20` |

---

## Database & Collection Operations

### Database Operations
```javascript
// Show databases
show dbs

// Use database (creates if doesn't exist)
use myapp

// Current database
db.getName()

// Database statistics
db.stats()

// Drop database
db.dropDatabase()
```

### Collection Operations
```javascript
// Show collections
show collections
// or
db.listCollections()

// Create collection
db.createCollection("users")

// Create collection with options
db.createCollection("users", {
    validator: {
        $jsonSchema: {
            bsonType: "object",
            required: ["username", "email"],
            properties: {
                username: { bsonType: "string" },
                email: { bsonType: "string", pattern: "^[^@]+@[^@]+\.[^@]+$" }
            }
        }
    }
})

// Collection statistics
db.users.stats()

// Drop collection
db.users.drop()

// Rename collection
db.users.renameCollection("customers")
```

---

## Document Operations (CRUD)

### Insert Operations
```javascript
// Insert single document
db.users.insertOne({
    username: "john_doe",
    email: "john@example.com",
    age: 30,
    tags: ["developer", "javascript"],
    profile: {
        firstName: "John",
        lastName: "Doe"
    },
    createdAt: new Date()
})

// Insert multiple documents
db.users.insertMany([
    { username: "jane_doe", email: "jane@example.com", age: 25 },
    { username: "bob_smith", email: "bob@example.com", age: 35 }
])

// Insert with custom _id
db.users.insertOne({
    _id: "custom_id_123",
    username: "admin",
    role: "administrator"
})
```

### Read Operations
```javascript
// Find all documents
db.users.find()

// Find with pretty formatting
db.users.find().pretty()

// Find one document
db.users.findOne({ username: "john_doe" })

// Find with projection (select specific fields)
db.users.find({}, { username: 1, email: 1, _id: 0 })

// Find with conditions
db.users.find({ age: { $gte: 25 } })

// Find with multiple conditions
db.users.find({ 
    age: { $gte: 25, $lt: 40 },
    "profile.firstName": "John"
})

// Count documents
db.users.countDocuments()
db.users.countDocuments({ age: { $gte: 18 } })

// Limit and skip
db.users.find().limit(10)
db.users.find().skip(20).limit(10)

// Sort
db.users.find().sort({ age: 1 })      // Ascending
db.users.find().sort({ age: -1 })     // Descending
db.users.find().sort({ age: -1, username: 1 })
```

### Update Operations
```javascript
// Update one document
db.users.updateOne(
    { username: "john_doe" },
    { $set: { age: 31, updatedAt: new Date() } }
)

// Update multiple documents
db.users.updateMany(
    { age: { $lt: 18 } },
    { $set: { status: "minor" } }
)

// Replace entire document
db.users.replaceOne(
    { username: "john_doe" },
    { 
        username: "john_doe",
        email: "john.doe@example.com",
        age: 31,
        updatedAt: new Date()
    }
)

// Upsert (update or insert)
db.users.updateOne(
    { username: "new_user" },
    { 
        $set: { email: "new@example.com" },
        $setOnInsert: { createdAt: new Date() }
    },
    { upsert: true }
)

// Update operators
db.users.updateOne(
    { username: "john_doe" },
    {
        $set: { "profile.city": "NYC" },        // Set field
        $unset: { temporaryField: "" },         // Remove field
        $inc: { loginCount: 1 },                // Increment number
        $push: { tags: "mongodb" },             // Add to array
        $pull: { tags: "beginner" },            // Remove from array
        $addToSet: { skills: "javascript" },    // Add to array if not exists
        $rename: { "oldField": "newField" }     // Rename field
    }
)
```

### Delete Operations
```javascript
// Delete one document
db.users.deleteOne({ username: "john_doe" })

// Delete multiple documents
db.users.deleteMany({ age: { $lt: 18 } })

// Delete all documents in collection
db.users.deleteMany({})
```

---

## Query Operations

### Comparison Operators
```javascript
// Basic comparison
db.users.find({ age: 25 })                    // Equal
db.users.find({ age: { $ne: 25 } })           // Not equal
db.users.find({ age: { $gt: 25 } })           // Greater than
db.users.find({ age: { $gte: 25 } })          // Greater than or equal
db.users.find({ age: { $lt: 30 } })           // Less than
db.users.find({ age: { $lte: 30 } })          // Less than or equal
db.users.find({ age: { $in: [25, 30, 35] } }) // In array
db.users.find({ age: { $nin: [25, 30] } })    // Not in array
```

### Logical Operators
```javascript
// AND (implicit)
db.users.find({ age: { $gte: 25 }, status: "active" })

// AND (explicit)
db.users.find({
    $and: [
        { age: { $gte: 25 } },
        { status: "active" }
    ]
})

// OR
db.users.find({
    $or: [
        { age: { $lt: 18 } },
        { age: { $gt: 65 } }
    ]
})

// NOT
db.users.find({ age: { $not: { $gte: 18 } } })

// NOR
db.users.find({
    $nor: [
        { status: "inactive" },
        { age: { $lt: 18 } }
    ]
})
```

### Element Operators
```javascript
// Field exists
db.users.find({ phone: { $exists: true } })

// Field type
db.users.find({ age: { $type: "int" } })
db.users.find({ age: { $type: "number" } })

// Array size
db.users.find({ tags: { $size: 3 } })

// All elements match
db.users.find({ tags: { $all: ["developer", "javascript"] } })
```

### Array Operators
```javascript
// Element in array matches condition
db.posts.find({ "comments.author": "john_doe" })

// Array element matches multiple conditions
db.posts.find({
    comments: {
        $elemMatch: {
            author: "john_doe",
            rating: { $gte: 5 }
        }
    }
})

// Match any array element
db.users.find({ tags: "developer" })

// Match all array elements
db.users.find({ "scores.$": { $gte: 80 } })
```

### Regular Expressions
```javascript
// Basic regex
db.users.find({ username: /^john/ })              // Starts with 'john'
db.users.find({ email: /@gmail\.com$/ })          // Ends with '@gmail.com'
db.users.find({ username: /john/i })              // Case insensitive

// Using $regex
db.users.find({ 
    username: { 
        $regex: "john", 
        $options: "i" 
    } 
})

// Text search (requires text index)
db.posts.find({ $text: { $search: "mongodb database" } })
```

### Geospatial Queries
```javascript
// Create 2dsphere index
db.places.createIndex({ location: "2dsphere" })

// Find near point
db.places.find({
    location: {
        $near: {
            $geometry: { type: "Point", coordinates: [-73.9667, 40.78] },
            $maxDistance: 1000  // meters
        }
    }
})

// Find within polygon
db.places.find({
    location: {
        $geoWithin: {
            $geometry: {
                type: "Polygon",
                coordinates: [[[0,0], [0,5], [5,5], [5,0], [0,0]]]
            }
        }
    }
})
```

---

## Aggregation Framework

### Basic Aggregation
```javascript
// Basic aggregation pipeline
db.users.aggregate([
    { $match: { age: { $gte: 18 } } },
    { $group: { _id: "$status", count: { $sum: 1 } } },
    { $sort: { count: -1 } }
])

// Count by field
db.users.aggregate([
    { $group: { _id: "$department", count: { $sum: 1 } } }
])

// Average calculation
db.users.aggregate([
    { $group: { _id: "$department", avgAge: { $avg: "$age" } } }
])
```

### Aggregation Stages

#### $match - Filtering
```javascript
db.orders.aggregate([
    { $match: { status: "completed", amount: { $gte: 100 } } }
])
```

#### $group - Grouping
```javascript
// Group with multiple accumulators
db.orders.aggregate([
    {
        $group: {
            _id: "$customerId",
            totalAmount: { $sum: "$amount" },
            avgAmount: { $avg: "$amount" },
            orderCount: { $sum: 1 },
            maxAmount: { $max: "$amount" },
            minAmount: { $min: "$amount" },
            orders: { $push: "$_id" },              // Array of all order IDs
            firstOrder: { $first: "$createdAt" },
            lastOrder: { $last: "$createdAt" }
        }
    }
])
```

#### $project - Field Selection and Transformation
```javascript
db.users.aggregate([
    {
        $project: {
            username: 1,
            email: 1,
            fullName: { $concat: ["$firstName", " ", "$lastName"] },
            age: 1,
            isAdult: { $gte: ["$age", 18] },
            yearBorn: { $subtract: [2023, "$age"] }
        }
    }
])
```

#### $lookup - Left Outer Join
```javascript
db.orders.aggregate([
    {
        $lookup: {
            from: "customers",
            localField: "customerId",
            foreignField: "_id",
            as: "customer"
        }
    },
    {
        $unwind: "$customer"  // Convert array to object
    }
])

// Advanced lookup with pipeline
db.orders.aggregate([
    {
        $lookup: {
            from: "products",
            let: { orderItems: "$items" },
            pipeline: [
                { $match: { $expr: { $in: ["$_id", "$$orderItems.productId"] } } }
            ],
            as: "products"
        }
    }
])
```

#### $unwind - Array Deconstruction
```javascript
db.posts.aggregate([
    { $unwind: "$tags" },
    { $group: { _id: "$tags", count: { $sum: 1 } } },
    { $sort: { count: -1 } }
])

// Preserve null and empty arrays
db.posts.aggregate([
    { $unwind: { path: "$tags", preserveNullAndEmptyArrays: true } }
])
```

#### $sort and $limit
```javascript
db.users.aggregate([
    { $sort: { age: -1, username: 1 } },
    { $limit: 10 }
])
```

#### $skip
```javascript
// Pagination
db.users.aggregate([
    { $skip: 20 },
    { $limit: 10 }
])
```

### Advanced Aggregation
```javascript
// Complex pipeline example
db.orders.aggregate([
    // Filter recent orders
    { $match: { createdAt: { $gte: new Date("2023-01-01") } } },
    
    // Lookup customer information
    {
        $lookup: {
            from: "customers",
            localField: "customerId",
            foreignField: "_id",
            as: "customer"
        }
    },
    { $unwind: "$customer" },
    
    // Group by customer location
    {
        $group: {
            _id: "$customer.city",
            totalRevenue: { $sum: "$amount" },
            orderCount: { $sum: 1 },
            avgOrderValue: { $avg: "$amount" },
            customers: { $addToSet: "$customerId" }
        }
    },
    
    // Add calculated fields
    {
        $project: {
            city: "$_id",
            totalRevenue: 1,
            orderCount: 1,
            avgOrderValue: { $round: ["$avgOrderValue", 2] },
            customerCount: { $size: "$customers" },
            revenuePerCustomer: { 
                $round: [{ $divide: ["$totalRevenue", { $size: "$customers" }] }, 2] 
            }
        }
    },
    
    // Sort by revenue
    { $sort: { totalRevenue: -1 } },
    
    // Top 10 cities
    { $limit: 10 }
])
```

### Aggregation Operators
```javascript
// Date operators
db.orders.aggregate([
    {
        $project: {
            year: { $year: "$createdAt" },
            month: { $month: "$createdAt" },
            dayOfWeek: { $dayOfWeek: "$createdAt" },
            formattedDate: { $dateToString: { format: "%Y-%m-%d", date: "$createdAt" } }
        }
    }
])

// String operators
db.users.aggregate([
    {
        $project: {
            username: 1,
            usernameUpper: { $toUpper: "$username" },
            usernameLower: { $toLower: "$username" },
            usernameLength: { $strLenCP: "$username" },
            emailDomain: { $substr: ["$email", { $indexOfCP: ["$email", "@"] }, -1] }
        }
    }
])

// Conditional operators
db.users.aggregate([
    {
        $project: {
            username: 1,
            age: 1,
            category: {
                $switch: {
                    branches: [
                        { case: { $lt: ["$age", 18] }, then: "Minor" },
                        { case: { $lt: ["$age", 65] }, then: "Adult" },
                        { case: { $gte: ["$age", 65] }, then: "Senior" }
                    ],
                    default: "Unknown"
                }
            }
        }
    }
])
```

---

## Indexing

### Basic Indexes
```javascript
// Single field index
db.users.createIndex({ username: 1 })        // Ascending
db.users.createIndex({ age: -1 })             // Descending

// Compound index
db.users.createIndex({ username: 1, age: -1 })

// Unique index
db.users.createIndex({ email: 1 }, { unique: true })

// Sparse index (only documents with the field)
db.users.createIndex({ phone: 1 }, { sparse: true })

// Partial index
db.users.createIndex(
    { age: 1 },
    { partialFilterExpression: { age: { $gte: 18 } } }
)

// TTL index (time to live)
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 })
```

### Text Indexes
```javascript
// Single field text index
db.posts.createIndex({ title: "text" })

// Multiple field text index
db.posts.createIndex({ title: "text", content: "text" })

// Text index with weights
db.posts.createIndex(
    { title: "text", content: "text" },
    { weights: { title: 10, content: 1 } }
)

// Text search
db.posts.find({ $text: { $search: "mongodb database" } })

// Text search with score
db.posts.find(
    { $text: { $search: "mongodb" } },
    { score: { $meta: "textScore" } }
).sort({ score: { $meta: "textScore" } })
```

### Geospatial Indexes
```javascript
// 2dsphere index for GeoJSON
db.places.createIndex({ location: "2dsphere" })

// 2d index for legacy coordinates
db.places.createIndex({ location: "2d" })

// Compound geospatial index
db.places.createIndex({ location: "2dsphere", category: 1 })
```

### Index Management
```javascript
// List all indexes
db.users.getIndexes()

// Get index usage statistics
db.users.aggregate([{ $indexStats: {} }])

// Drop index
db.users.dropIndex({ username: 1 })
db.users.dropIndex("username_1")

// Drop all indexes (except _id)
db.users.dropIndexes()

// Reindex collection
db.users.reIndex()

// Explain query execution
db.users.find({ username: "john_doe" }).explain("executionStats")
```

### Index Performance
```javascript
// Analyze query performance
db.users.find({ age: { $gte: 25 }, status: "active" }).explain("executionStats")

// Hint to use specific index
db.users.find({ age: 25 }).hint({ age: 1 })

// Get current operations
db.currentOp()

// Index intersection
db.users.createIndex({ age: 1 })
db.users.createIndex({ status: 1 })
// MongoDB can use both indexes for: { age: 25, status: "active" }
```

---

## Data Modeling

### Document Structure
```javascript
// Embedded documents
{
    _id: ObjectId("..."),
    username: "john_doe",
    profile: {
        firstName: "John",
        lastName: "Doe",
        address: {
            street: "123 Main St",
            city: "NYC",
            zipCode: "10001"
        }
    },
    tags: ["developer", "javascript", "mongodb"]
}

// References
{
    _id: ObjectId("..."),
    username: "john_doe",
    profileId: ObjectId("..."),  // Reference to profiles collection
    groupIds: [                  // Array of references
        ObjectId("..."),
        ObjectId("...")
    ]
}
```

### One-to-One Relationships
```javascript
// Embedded approach (preferred for closely related data)
{
    _id: ObjectId("..."),
    username: "john_doe",
    profile: {
        firstName: "John",
        lastName: "Doe",
        bio: "Software developer"
    }
}

// Reference approach (for large or rarely accessed data)
// users collection
{ _id: ObjectId("user1"), username: "john_doe", profileId: ObjectId("profile1") }

// profiles collection
{ _id: ObjectId("profile1"), firstName: "John", lastName: "Doe", bio: "..." }
```

### One-to-Many Relationships
```javascript
// Embedded approach (few related documents)
{
    _id: ObjectId("..."),
    title: "Blog Post",
    content: "...",
    comments: [
        { author: "user1", text: "Great post!", createdAt: new Date() },
        { author: "user2", text: "Thanks for sharing", createdAt: new Date() }
    ]
}

// Reference approach (many related documents)
// posts collection
{ _id: ObjectId("post1"), title: "Blog Post", content: "..." }

// comments collection
{ _id: ObjectId("..."), postId: ObjectId("post1"), author: "user1", text: "Great post!" }
```

### Many-to-Many Relationships
```javascript
// Array of references in both collections
// users collection
{
    _id: ObjectId("user1"),
    username: "john_doe",
    groupIds: [ObjectId("group1"), ObjectId("group2")]
}

// groups collection
{
    _id: ObjectId("group1"),
    name: "Developers",
    memberIds: [ObjectId("user1"), ObjectId("user2")]
}

// Separate junction collection (for additional metadata)
// memberships collection
{
    _id: ObjectId("..."),
    userId: ObjectId("user1"),
    groupId: ObjectId("group1"),
    joinedAt: new Date(),
    role: "admin"
}
```

### Schema Validation
```javascript
// JSON Schema validation
db.createCollection("users", {
    validator: {
        $jsonSchema: {
            bsonType: "object",
            required: ["username", "email", "age"],
            properties: {
                username: {
                    bsonType: "string",
                    minLength: 3,
                    maxLength: 30,
                    description: "must be a string between 3-30 characters"
                },
                email: {
                    bsonType: "string",
                    pattern: "^[^@]+@[^@]+\.[^@]+$",
                    description: "must be a valid email"
                },
                age: {
                    bsonType: "int",
                    minimum: 0,
                    maximum: 120,
                    description: "must be an integer between 0-120"
                }
            }
        }
    }
})

// Update validation rules
db.runCommand({
    collMod: "users",
    validator: {
        $jsonSchema: {
            // updated schema
        }
    }
})
```

---

## Replication

### Replica Set Setup
```javascript
// Initialize replica set
rs.initiate({
    _id: "rs0",
    members: [
        { _id: 0, host: "mongodb1:27017" },
        { _id: 1, host: "mongodb2:27017" },
        { _id: 2, host: "mongodb3:27017" }
    ]
})

// Add member to replica set
rs.add("mongodb4:27017")

// Add arbiter
rs.addArb("mongodb5:27017")

// Remove member
rs.remove("mongodb4:27017")

// Check replica set status
rs.status()

// Check replica set configuration
rs.conf()

// Step down primary (for maintenance)
rs.stepDown(60)  // seconds
```

### Read Preferences
```javascript
// Read from primary only (default)
db.users.find().readPref("primary")

// Read from secondary if available
db.users.find().readPref("primaryPreferred")

// Read from secondary only
db.users.find().readPref("secondary")

// Read from secondary, fallback to primary
db.users.find().readPref("secondaryPreferred")

// Read from nearest member
db.users.find().readPref("nearest")
```

### Write Concerns
```javascript
// Wait for acknowledgment from majority
db.users.insertOne(
    { username: "john_doe" },
    { writeConcern: { w: "majority", j: true, wtimeout: 5000 } }
)

// Write concern options:
// w: 1 (default) - primary only
// w: "majority" - majority of replica set
// w: 3 - wait for 3 members
// j: true - wait for journal sync
// wtimeout: milliseconds to wait
```

---

## Sharding

### Shard Key Selection
```javascript
// Good shard keys have:
// 1. High cardinality
// 2. Even distribution
// 3. Query isolation

// Range-based sharding
sh.shardCollection("myapp.users", { userId: 1 })

// Hash-based sharding (better distribution)
sh.shardCollection("myapp.users", { userId: "hashed" })

// Compound shard key
sh.shardCollection("myapp.orders", { customerId: 1, orderDate: 1 })
```

### Sharding Administration
```javascript
// Enable sharding on database
sh.enableSharding("myapp")

// Check sharding status
sh.status()

// List shards
sh.listShards()

// Add shard
sh.addShard("rs1/mongodb6:27017,mongodb7:27017,mongodb8:27017")

// Remove shard
sh.removeShard("shard0000")

// Check balancer status
sh.getBalancerState()

// Start/stop balancer
sh.startBalancer()
sh.stopBalancer()
```

---

## Administration

### User Management
```javascript
// Create admin user
use admin
db.createUser({
    user: "admin",
    pwd: "password",
    roles: ["userAdminAnyDatabase", "dbAdminAnyDatabase", "readWriteAnyDatabase"]
})

// Create database-specific user
use myapp
db.createUser({
    user: "appuser",
    pwd: "password",
    roles: [
        { role: "readWrite", db: "myapp" },
        { role: "read", db: "analytics" }
    ]
})

// Update user
db.updateUser("appuser", {
    pwd: "newpassword",
    roles: [{ role: "readWrite", db: "myapp" }]
})

// List users
db.getUsers()

// Drop user
db.dropUser("appuser")

// Grant role
db.grantRolesToUser("appuser", [{ role: "dbAdmin", db: "myapp" }])

// Revoke role
db.revokeRolesFromUser("appuser", [{ role: "dbAdmin", db: "myapp" }])
```

### Database Statistics
```javascript
// Database stats
db.stats()

// Collection stats
db.users.stats()

// Server status
db.serverStatus()

// Current operations
db.currentOp()

// Kill operation
db.killOp(12345)  // operation ID

// Profiler
db.setProfilingLevel(1, { slowms: 100 })  // Profile slow operations
db.system.profile.find().sort({ ts: -1 }).limit(5)  // Recent slow operations
db.setProfilingLevel(0)  // Disable profiler
```

### Backup and Restore
```bash
# mongodump - backup
mongodump --host localhost:27017 --db myapp --out /backup/

# mongodump with authentication
mongodump --host localhost:27017 --username admin --password --authenticationDatabase admin --db myapp --out /backup/

# mongodump specific collection
mongodump --db myapp --collection users --out /backup/

# mongorestore
mongorestore --host localhost:27017 --db myapp /backup/myapp/

# mongorestore with drop
mongorestore --host localhost:27017 --db myapp --drop /backup/myapp/

# Export to JSON
mongoexport --db myapp --collection users --out users.json

# Import from JSON
mongoimport --db myapp --collection users --file users.json
```

---

## Performance Optimization

### Query Performance
```javascript
// Use explain() to analyze queries
db.users.find({ age: { $gte: 25 } }).explain("executionStats")

// Key metrics to monitor:
// - totalDocsExamined vs totalDocsReturned
// - executionTimeMillis
// - indexesUsed

// Create appropriate indexes
db.users.createIndex({ age: 1 })
db.users.createIndex({ status: 1, age: 1 })  // Compound index

// Use projection to limit fields
db.users.find({ age: { $gte: 25 } }, { username: 1, email: 1, _id: 0 })

// Limit results
db.users.find().limit(10)
```

### Aggregation Performance
```javascript
// Put $match early in pipeline
db.orders.aggregate([
    { $match: { status: "completed" } },  // Filter first
    { $lookup: { ... } },                 // Then join
    { $group: { ... } }                   // Then group
])

// Use indexes for $match and $sort
db.orders.createIndex({ status: 1, createdAt: -1 })

// Avoid $unwind on large arrays
// Instead of:
db.posts.aggregate([
    { $unwind: "$comments" },
    { $group: { _id: "$_id", commentCount: { $sum: 1 } } }
])

// Use:
db.posts.aggregate([
    { $project: { commentCount: { $size: "$comments" } } }
])
```

### Connection Pooling
```javascript
// MongoDB drivers automatically pool connections
// Typical settings:
{
    maxPoolSize: 100,           // Maximum connections
    minPoolSize: 5,             // Minimum connections
    maxIdleTimeMS: 30000,       // Close idle connections after 30s
    waitQueueTimeoutMS: 5000,   // Wait 5s for available connection
    serverSelectionTimeoutMS: 5000
}
```

### Memory Management
```bash
# Monitor memory usage
db.serverStatus().mem

# WiredTiger cache size (default: 50% of RAM - 1GB)
storage:
  wiredTiger:
    engineConfig:
      cacheSizeGB: 4

# Monitor cache usage
db.serverStatus().wiredTiger.cache
```

---

## Security

### Authentication
```javascript
// Enable authentication in mongod.conf
security:
  authorization: enabled

// Create first admin user
use admin
db.createUser({
    user: "admin",
    pwd: "strongpassword",
    roles: ["userAdminAnyDatabase"]
})

// Built-in roles:
// - read, readWrite
// - dbAdmin, userAdmin
// - readAnyDatabase, readWriteAnyDatabase
// - dbAdminAnyDatabase, userAdminAnyDatabase
// - root, __system
```

### SSL/TLS
```yaml
# mongod.conf
net:
  ssl:
    mode: requireSSL
    PEMKeyFile: /path/to/mongodb.pem
    CAFile: /path/to/ca.pem
```

### Field-Level Encryption
```javascript
// Client-side field level encryption (MongoDB 4.2+)
const clientEncryption = new ClientEncryption(keyVault, {
    keyVaultNamespace: 'encryption.__keyVault',
    kmsProviders: {
        local: { key: masterKey }
    }
})

// Create data encryption key
const keyId = await clientEncryption.createDataKey('local')

// Encrypt field
const encryptedSSN = await clientEncryption.encrypt(
    '123-45-6789',
    {
        keyId: keyId,
        algorithm: 'AEAD_AES_256_CBC_HMAC_SHA_512-Deterministic'
    }
)
```

### Network Security
```yaml
# mongod.conf
net:
  bindIp: 127.0.0.1,10.0.0.100  # Specific IPs only
  port: 27017

# Firewall rules
# Only allow access from application servers
iptables -A INPUT -p tcp --dport 27017 -s 10.0.1.0/24 -j ACCEPT
```

---

## GridFS

### Storing Files
```javascript
// Using MongoDB shell
use myapp

// Store file
db.fs.files.insertOne({
    filename: "document.pdf",
    contentType: "application/pdf",
    metadata: { author: "John Doe", category: "reports" }
})

// Using Node.js driver
const { GridFSBucket } = require('mongodb')

const bucket = new GridFSBucket(db, { bucketName: 'uploads' })

// Upload file
const uploadStream = bucket.openUploadStream('document.pdf', {
    metadata: { author: 'John Doe' }
})

// Download file
const downloadStream = bucket.openDownloadStreamByName('document.pdf')

// Find files
const files = await db.collection('uploads.files').find({ 
    filename: 'document.pdf' 
}).toArray()

// Delete file
await bucket.delete(fileId)
```

---

## Integration Patterns

### Node.js Integration
```javascript
const { MongoClient } = require('mongodb')

const client = new MongoClient('mongodb://localhost:27017', {
    maxPoolSize: 10,
    serverSelectionTimeoutMS: 5000,
    socketTimeoutMS: 45000,
})

async function connectToMongoDB() {
    try {
        await client.connect()
        const db = client.db('myapp')
        return db
    } catch (error) {
        console.error('MongoDB connection error:', error)
        throw error
    }
}

// CRUD operations
async function createUser(userData) {
    const db = await connectToMongoDB()
    const result = await db.collection('users').insertOne(userData)
    return result.insertedId
}

async function getUser(userId) {
    const db = await connectToMongoDB()
    return await db.collection('users').findOne({ _id: userId })
}

// Transactions (MongoDB 4.0+)
async function transferFunds(fromAccount, toAccount, amount) {
    const session = client.startSession()
    
    try {
        const transactionResults = await session.withTransaction(async () => {
            const accounts = client.db('banking').collection('accounts')
            
            await accounts.updateOne(
                { _id: fromAccount },
                { $inc: { balance: -amount } },
                { session }
            )
            
            await accounts.updateOne(
                { _id: toAccount },
                { $inc: { balance: amount } },
                { session }
            )
        })
        
        return transactionResults
    } finally {
        await session.endSession()
    }
}
```

### Python Integration (PyMongo)
```python
from pymongo import MongoClient
from bson import ObjectId

# Connection
client = MongoClient('mongodb://localhost:27017/', maxPoolSize=50)
db = client.myapp
users = db.users

# CRUD operations
def create_user(user_data):
    result = users.insert_one(user_data)
    return result.inserted_id

def get_user(user_id):
    return users.find_one({'_id': ObjectId(user_id)})

def update_user(user_id, update_data):
    result = users.update_one(
        {'_id': ObjectId(user_id)},
        {'$set': update_data}
    )
    return result.modified_count

# Aggregation
def get_user_stats():
    pipeline = [
        {'$group': {
            '_id': '$status',
            'count': {'$sum': 1},
            'avg_age': {'$avg': '$age'}
        }},
        {'$sort': {'count': -1}}
    ]
    return list(users.aggregate(pipeline))

# Error handling
try:
    create_user({'username': 'john_doe', 'email': 'john@example.com'})
except Exception as e:
    print(f"Error creating user: {e}")
```

### Spring Data MongoDB
```java
// Entity
@Document(collection = "users")
public class User {
    @Id
    private String id;
    
    @Field("username")
    private String username;
    
    @Field("email")
    private String email;
    
    @Field("created_at")
    private LocalDateTime createdAt;
    
    // getters and setters
}

// Repository
@Repository
public interface UserRepository extends MongoRepository<User, String> {
    List<User> findByUsername(String username);
    List<User> findByAgeGreaterThan(int age);
    
    @Query("{ 'email' : { $regex: ?0, $options: 'i' } }")
    List<User> findByEmailRegex(String email);
    
    @Aggregation(pipeline = {
        "{ $group: { _id: '$status', count: { $sum: 1 } } }",
        "{ $sort: { count: -1 } }"
    })
    List<UserStats> getUserStatsByStatus();
}

// Service
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    
    public User createUser(User user) {
        user.setCreatedAt(LocalDateTime.now());
        return userRepository.save(user);
    }
    
    public List<User> findActiveUsers() {
        return userRepository.findByStatus("active");
    }
}
```

---

## Performance Tuning Tips

1. **Index Strategy**: Create indexes for frequently queried fields
2. **Query Optimization**: Use explain() to analyze query performance
3. **Document Structure**: Design documents for your query patterns
4. **Connection Pooling**: Configure appropriate pool sizes
5. **Read Preferences**: Use secondary reads for read-heavy workloads
6. **Aggregation Optimization**: Put $match stages early in pipelines
7. **Limit Results**: Always use limit() for large result sets
8. **Projection**: Only select fields you need
9. **Monitoring**: Use MongoDB Compass or ops manager
10. **Hardware**: Use SSDs and sufficient RAM for working set

---

## Resources
- [MongoDB Official Documentation](https://docs.mongodb.com/)
- [MongoDB University](https://university.mongodb.com/)
- [MongoDB Compass](https://www.mongodb.com/products/compass) - GUI tool
- [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) - Cloud database
- [Robo 3T](https://robomongo.org/) - MongoDB GUI
- [MongoDB Performance Best Practices](https://www.mongodb.com/blog/post/performance-best-practices-mongodb-data-modeling-and-memory-sizing)

---
*Comprehensive MongoDB reference for NoSQL document database operations. Contributions welcome!*