# 🗄️ Database Architecture: MongoDB & Mongoose

**Documented:** November 19, 2025
**Target:** React developers learning databases (no prior DB experience)
**Time Estimate:** 6-8 hours to fully understand
**Difficulty:** 🟡 Intermediate

---

## Welcome to Databases!

If you're coming from React and have never worked with databases, this guide will help you understand data persistence using MongoDB and Mongoose in LibreChat.

---

## Table of Contents

- [Mental Model: React State → Database](#mental-model-react-state--database)
- [What is MongoDB?](#what-is-mongodb)
- [What is Mongoose?](#what-is-mongoose)
- [Database Structure](#database-structure)
- [Schemas: Your TypeScript Interfaces with Validation](#schemas-your-typescript-interfaces-with-validation)
- [Relationships: Linking Data](#relationships-linking-data)
- [Queries: Finding Data](#queries-finding-data)
- [Aggregations: Complex Data Processing](#aggregations-complex-data-processing)
- [Indexes: Making Queries Fast](#indexes-making-queries-fast)
- [Data Modeling Best Practices](#data-modeling-best-practices)
- [Migrations: Changing Schemas](#migrations-changing-schemas)
- [Performance Optimization](#performance-optimization)
- [Common Patterns in LibreChat](#common-patterns-in-librechat)

---

## Mental Model: React State → Database

**If you understand React state, you already understand databases!**

| React Concept | Database Equivalent | Explanation |
|--------------|---------------------|-------------|
| **`useState`** | Database collection | Persistent state storage |
| **`localStorage`** | Database (but better) | Survives server restarts |
| **TypeScript interface** | Mongoose schema | Data structure definition |
| **Zod validation** | Mongoose validation | Data validation rules |
| **Array of objects** | Collection of documents | List of similar items |
| **`array.find()`** | `Model.findOne()` | Find single item |
| **`array.filter()`** | `Model.find()` | Find multiple items |
| **`array.map()`** | Aggregation pipeline | Transform data |
| **Nested objects** | Embedded documents | Related data |
| **Foreign keys** | References (ObjectId) | Link between collections |

**Key Differences:**

```javascript
// React: State lost on refresh
const [messages, setMessages] = useState([]);

// Database: State persists forever
const messages = await Message.find();
```

```javascript
// React: Manual validation
if (!text || text.length > 10000) {
  setError('Invalid text');
}

// Database: Automatic validation
const schema = new Schema({
  text: { type: String, required: true, maxlength: 10000 }
});
```

---

## What is MongoDB?

**MongoDB** is a **NoSQL** database. Instead of tables (SQL), it uses **collections** and **documents**.

### SQL vs NoSQL Comparison

**SQL (Relational):**
```sql
-- Tables with fixed columns
CREATE TABLE messages (
  id INT PRIMARY KEY,
  text VARCHAR(10000),
  user_id INT,
  created_at TIMESTAMP
);

-- Insert data
INSERT INTO messages (text, user_id) VALUES ('Hello', 123);

-- Query data
SELECT * FROM messages WHERE user_id = 123;
```

**MongoDB (NoSQL):**
```javascript
// Collections with flexible documents (JSON-like)
{
  _id: ObjectId("507f1f77bcf86cd799439011"),
  text: "Hello",
  userId: ObjectId("507f191e810c19729de860ea"),
  createdAt: ISODate("2025-11-19T10:00:00Z")
}

// Insert data
await Message.create({ text: 'Hello', userId: '123' });

// Query data
await Message.find({ userId: '123' });
```

### Why MongoDB for LibreChat?

✅ **Advantages:**
- **Flexible schema:** Easy to add new fields
- **JSON-like documents:** Works naturally with JavaScript
- **Fast reads:** Good for chat message retrieval
- **Nested data:** Store related data together
- **Scalable:** Can handle millions of messages

❌ **When NOT to use MongoDB:**
- Complex joins (use SQL instead)
- Strict ACID transactions (banking apps)
- Heavily relational data (use PostgreSQL)

### MongoDB Terminology

| SQL Term | MongoDB Term | React Analogy |
|----------|-------------|---------------|
| Database | Database | Project |
| Table | Collection | Array |
| Row | Document | Object in array |
| Column | Field | Object property |
| Primary Key | `_id` field | Unique ID |
| Foreign Key | Reference (ObjectId) | ID linking objects |
| Join | `$lookup` or `populate` | Combining data |

---

## What is Mongoose?

**Mongoose** is an **ODM** (Object-Document Mapper) for MongoDB.

**Think of it as:**
- TypeScript interfaces + validation
- Helper methods for queries
- Middleware (like React hooks for data)

**Without Mongoose (raw MongoDB):**
```javascript
const db = await MongoClient.connect('mongodb://localhost');
const collection = db.collection('messages');

// Manual validation
if (!text || typeof text !== 'string') {
  throw new Error('Invalid text');
}

await collection.insertOne({
  text,
  userId,
  createdAt: new Date()
});
```

**With Mongoose:**
```javascript
// Define schema with validation
const messageSchema = new Schema({
  text: { type: String, required: true, maxlength: 10000 },
  userId: { type: ObjectId, required: true }
}, { timestamps: true }); // Auto-add createdAt, updatedAt

const Message = model('Message', messageSchema);

// Validation happens automatically
await Message.create({ text, userId }); // Throws error if invalid
```

**React + TypeScript Analogy:**

```typescript
// React: Define interface (compile-time only)
interface Message {
  text: string;
  userId: string;
}

// Mongoose: Define schema (runtime validation + database)
const messageSchema = new Schema({
  text: { type: String, required: true },
  userId: { type: String, required: true }
});
```

---

## Database Structure

### LibreChat Collections

```
librechat (database)
├── users
│   └── { _id, name, email, password, createdAt, ... }
│
├── conversations
│   └── { _id, userId, title, model, createdAt, ... }
│
├── messages
│   └── { _id, conversationId, userId, role, text, ... }
│
├── presets
│   └── { _id, userId, name, model, parameters, ... }
│
├── agents
│   └── { _id, userId, name, description, tools, ... }
│
└── files
    └── { _id, userId, filename, mimetype, path, ... }
```

**Visualization:**

```
┌─────────────┐
│    User     │
└──────┬──────┘
       │ has many
       ▼
┌─────────────────┐
│  Conversations  │
└──────┬──────────┘
       │ has many
       ▼
┌─────────────┐
│  Messages   │
└─────────────┘
```

---

## Schemas: Your TypeScript Interfaces with Validation

### Basic Schema Example

**File:** `api/models/User.js`

```javascript
const mongoose = require('mongoose');
const { Schema } = mongoose;

/**
 * User Schema
 * Represents a user account
 */
const userSchema = new Schema(
  {
    // ===========================
    // Basic Fields
    // ===========================

    name: {
      type: String,
      required: [true, 'Name is required'],
      trim: true,
      minlength: [2, 'Name must be at least 2 characters'],
      maxlength: [50, 'Name must be less than 50 characters']
    },

    email: {
      type: String,
      required: [true, 'Email is required'],
      unique: true,
      lowercase: true,
      trim: true,
      match: [/^\S+@\S+\.\S+$/, 'Invalid email format']
    },

    password: {
      type: String,
      required: [true, 'Password is required'],
      minlength: [8, 'Password must be at least 8 characters'],
      select: false // Don't return password by default
    },

    // ===========================
    // Optional Fields
    // ===========================

    avatar: {
      type: String,
      default: null
    },

    role: {
      type: String,
      enum: ['user', 'admin'],
      default: 'user'
    },

    isVerified: {
      type: Boolean,
      default: false
    },

    lastLoginAt: Date
  },
  {
    // ===========================
    // Schema Options
    // ===========================

    timestamps: true, // Auto-add createdAt, updatedAt
    toJSON: { virtuals: true }, // Include virtual properties
    toObject: { virtuals: true }
  }
);

const User = mongoose.model('User', userSchema);

module.exports = User;
```

**React + TypeScript Equivalent:**

```typescript
// React: TypeScript interface (no validation)
interface User {
  name: string;
  email: string;
  password: string;
  avatar?: string | null;
  role: 'user' | 'admin';
  isVerified: boolean;
  lastLoginAt?: Date;
  createdAt: Date;
  updatedAt: Date;
}

// Mongoose: Schema with runtime validation
const userSchema = new Schema({
  name: { type: String, required: true, minlength: 2, maxlength: 50 },
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true, minlength: 8 },
  avatar: { type: String, default: null },
  role: { type: String, enum: ['user', 'admin'], default: 'user' },
  isVerified: { type: Boolean, default: false },
  lastLoginAt: Date
}, { timestamps: true });
```

### Schema Field Types

```javascript
const schema = new Schema({
  // ===========================
  // Primitive Types
  // ===========================

  stringField: String,
  numberField: Number,
  booleanField: Boolean,
  dateField: Date,

  // ===========================
  // MongoDB Specific
  // ===========================

  objectIdField: Schema.Types.ObjectId,
  mixedField: Schema.Types.Mixed, // Any type (avoid if possible)
  bufferField: Buffer, // Binary data

  // ===========================
  // Arrays
  // ===========================

  arrayOfStrings: [String],
  arrayOfNumbers: [Number],
  arrayOfObjects: [{
    name: String,
    value: Number
  }],

  // ===========================
  // Nested Objects
  // ===========================

  nestedObject: {
    field1: String,
    field2: Number
  },

  // ===========================
  // With Validation
  // ===========================

  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    trim: true,
    match: /^\S+@\S+\.\S+$/
  },

  age: {
    type: Number,
    min: 0,
    max: 120,
    default: 18
  },

  status: {
    type: String,
    enum: ['active', 'inactive', 'banned'],
    default: 'active'
  }
});
```

### Complete Example: Message Schema

**File:** `api/models/Message.js`

```javascript
const mongoose = require('mongoose');
const { Schema } = mongoose;

const messageSchema = new Schema(
  {
    conversationId: {
      type: Schema.Types.ObjectId,
      ref: 'Conversation', // Reference to Conversation collection
      required: true,
      index: true // Create index for faster queries
    },

    userId: {
      type: Schema.Types.ObjectId,
      ref: 'User',
      required: true,
      index: true
    },

    role: {
      type: String,
      enum: ['user', 'assistant', 'system'],
      required: true
    },

    text: {
      type: String,
      required: true,
      maxlength: 10000
    },

    model: {
      type: String,
      enum: ['gpt-4', 'gpt-3.5-turbo', 'claude-3-sonnet', 'gemini-pro']
    },

    // Metadata
    metadata: {
      tokenCount: Number,
      finishReason: String,
      processingTime: Number
    },

    // Files attached to message
    fileIds: [{
      type: Schema.Types.ObjectId,
      ref: 'File'
    }],

    // Editing
    edited: {
      type: Boolean,
      default: false
    },

    editedAt: Date,

    // Error tracking
    error: String
  },
  {
    timestamps: true
  }
);

// ===========================
// Indexes (for performance)
// ===========================

messageSchema.index({ conversationId: 1, createdAt: -1 });
messageSchema.index({ userId: 1 });

// ===========================
// Virtual Properties
// ===========================

messageSchema.virtual('isEdited').get(function() {
  return this.edited && this.editedAt != null;
});

// ===========================
// Instance Methods
// ===========================

messageSchema.methods.toClient = function() {
  return {
    id: this._id.toString(),
    conversationId: this.conversationId.toString(),
    role: this.role,
    text: this.text,
    model: this.model,
    edited: this.edited,
    editedAt: this.editedAt,
    createdAt: this.createdAt
  };
};

// ===========================
// Static Methods
// ===========================

messageSchema.statics.findByConversation = function(conversationId, options = {}) {
  const { limit = 50, skip = 0 } = options;

  return this.find({ conversationId })
    .sort({ createdAt: 1 })
    .limit(limit)
    .skip(skip)
    .lean();
};

// ===========================
// Middleware (Hooks)
// ===========================

// Before save
messageSchema.pre('save', function(next) {
  if (this.text) {
    this.text = this.text.trim();
  }
  next();
});

// After save
messageSchema.post('save', async function(doc) {
  // Update conversation's lastMessageAt
  await mongoose.model('Conversation').findByIdAndUpdate(doc.conversationId, {
    lastMessageAt: new Date()
  });
});

const Message = mongoose.model('Message', messageSchema);

module.exports = Message;
```

---

## Relationships: Linking Data

### One-to-Many: References

**Example:** One User has many Conversations

```javascript
// User schema
const userSchema = new Schema({
  name: String,
  email: String
});

// Conversation schema
const conversationSchema = new Schema({
  userId: {
    type: Schema.Types.ObjectId,
    ref: 'User' // Reference to User collection
  },
  title: String
});

// Usage
const user = await User.create({ name: 'John', email: 'john@example.com' });

const conversation = await Conversation.create({
  userId: user._id, // Store user's ID
  title: 'My Conversation'
});

// Later, get conversation with user data
const conv = await Conversation.findById(conversationId)
  .populate('userId'); // Load user data

console.log(conv.userId.name); // 'John'
```

**React Analogy:**

```javascript
// React: Store ID and fetch separately
const [conversation, setConversation] = useState({ userId: '123', title: '...' });
const [user, setUser] = useState(null);

useEffect(() => {
  fetchUser(conversation.userId).then(setUser);
}, [conversation.userId]);

// Mongoose: Store ID and populate
const conversation = await Conversation.findById('...')
  .populate('userId');
// conversation.userId now has full user data
```

### Many-to-Many: Arrays of References

**Example:** Messages can have multiple Files, Files can be in multiple Messages

```javascript
// Message schema
const messageSchema = new Schema({
  text: String,
  fileIds: [{
    type: Schema.Types.ObjectId,
    ref: 'File'
  }]
});

// File schema
const fileSchema = new Schema({
  filename: String,
  path: String
});

// Usage
const file1 = await File.create({ filename: 'image.png', path: '/uploads/...' });
const file2 = await File.create({ filename: 'doc.pdf', path: '/uploads/...' });

const message = await Message.create({
  text: 'Check these files',
  fileIds: [file1._id, file2._id]
});

// Later, get message with files
const msg = await Message.findById(messageId)
  .populate('fileIds');

console.log(msg.fileIds[0].filename); // 'image.png'
console.log(msg.fileIds[1].filename); // 'doc.pdf'
```

### Embedded Documents

**When to embed:** Small, related data that's always accessed together

```javascript
// Instead of separate collection
const conversationSchema = new Schema({
  title: String,

  // Embed settings (not in separate collection)
  settings: {
    model: {
      type: String,
      default: 'gpt-4'
    },
    temperature: {
      type: Number,
      min: 0,
      max: 2,
      default: 0.7
    },
    maxTokens: {
      type: Number,
      default: 2000
    }
  }
});

// Usage
const conversation = await Conversation.create({
  title: 'My Chat',
  settings: {
    model: 'claude-3-sonnet',
    temperature: 0.8,
    maxTokens: 3000
  }
});

// Access embedded data
console.log(conversation.settings.model); // 'claude-3-sonnet'
```

**References vs Embedded:**

| Use References | Use Embedded |
|---------------|-------------|
| Large documents | Small documents |
| Accessed separately | Always accessed together |
| Shared across collections | Belongs to one parent |
| Example: Users, Conversations | Example: Settings, Metadata |

---

## Queries: Finding Data

### Basic Queries

```javascript
// ===========================
// Create
// ===========================

// Create one
const user = await User.create({
  name: 'John',
  email: 'john@example.com',
  password: 'hashedpassword'
});

// Create many
const users = await User.insertMany([
  { name: 'John', email: 'john@example.com' },
  { name: 'Jane', email: 'jane@example.com' }
]);

// ===========================
// Read (Find)
// ===========================

// Find all
const allUsers = await User.find();

// Find with filter
const admins = await User.find({ role: 'admin' });

// Find one
const user = await User.findOne({ email: 'john@example.com' });

// Find by ID
const user = await User.findById('507f1f77bcf86cd799439011');

// ===========================
// Update
// ===========================

// Update one
const updated = await User.findByIdAndUpdate(
  userId,
  { name: 'John Doe' },
  { new: true } // Return updated document
);

// Update many
await User.updateMany(
  { role: 'user' },
  { isVerified: true }
);

// ===========================
// Delete
// ===========================

// Delete one
await User.findByIdAndDelete(userId);

// Delete many
await User.deleteMany({ isVerified: false });
```

### Advanced Queries

**Comparison Operators:**

```javascript
// Greater than / Less than
await Message.find({
  createdAt: { $gte: yesterday } // Greater than or equal
});

await User.find({
  age: { $lt: 18 } // Less than
});

// In array
await Message.find({
  role: { $in: ['user', 'assistant'] }
});

// Not equal
await User.find({
  role: { $ne: 'admin' }
});

// Exists
await User.find({
  avatar: { $exists: true }
});
```

**Logical Operators:**

```javascript
// AND (implicit)
await Message.find({
  conversationId: '123',
  role: 'user'
});

// OR
await User.find({
  $or: [
    { email: 'john@example.com' },
    { email: 'jane@example.com' }
  ]
});

// AND + OR combined
await Message.find({
  conversationId: '123',
  $or: [
    { role: 'user' },
    { role: 'assistant' }
  ]
});

// NOT
await User.find({
  email: { $not: { $regex: /@test\.com$/ } }
});
```

**String Matching:**

```javascript
// Exact match
await User.find({ name: 'John' });

// Case-insensitive
await User.find({
  name: { $regex: 'john', $options: 'i' }
});

// Starts with
await User.find({
  email: { $regex: '^john' }
});

// Contains
await Message.find({
  text: { $regex: 'hello', $options: 'i' }
});
```

**Query Chaining:**

```javascript
const messages = await Message
  .find({ conversationId: '123' })
  .where('role').equals('user')
  .where('createdAt').gte(yesterday)
  .sort({ createdAt: -1 }) // -1 = descending
  .limit(20)
  .skip(0) // Pagination offset
  .select('text role createdAt') // Only these fields
  .populate('userId', 'name email')
  .lean(); // Return plain JS objects (faster)
```

**React Analogy:**

```javascript
// React: Array methods
const filtered = messages
  .filter(m => m.conversationId === '123')
  .filter(m => m.role === 'user')
  .filter(m => m.createdAt >= yesterday)
  .sort((a, b) => b.createdAt - a.createdAt)
  .slice(0, 20);

// Mongoose: Query methods
const filtered = await Message
  .find({ conversationId: '123' })
  .where('role').equals('user')
  .where('createdAt').gte(yesterday)
  .sort({ createdAt: -1 })
  .limit(20);
```

---

## Aggregations: Complex Data Processing

**Aggregation Pipeline:** Chain of operations to transform data

**React Analogy:** Like chaining `.map()`, `.filter()`, `.reduce()`

### Example 1: Count Messages Per User

```javascript
const stats = await Message.aggregate([
  // Stage 1: Match documents (filter)
  {
    $match: {
      createdAt: { $gte: lastWeek }
    }
  },

  // Stage 2: Group by userId
  {
    $group: {
      _id: '$userId',
      messageCount: { $sum: 1 },
      totalCharacters: {
        $sum: { $strLenCP: '$text' }
      }
    }
  },

  // Stage 3: Sort by message count
  {
    $sort: { messageCount: -1 }
  },

  // Stage 4: Limit to top 10
  {
    $limit: 10
  }
]);

// Result:
[
  { _id: 'user123', messageCount: 150, totalCharacters: 12000 },
  { _id: 'user456', messageCount: 120, totalCharacters: 9500 },
  // ...
]
```

**React Equivalent:**

```javascript
const stats = messages
  .filter(m => m.createdAt >= lastWeek) // $match
  .reduce((acc, m) => {
    const userId = m.userId;
    if (!acc[userId]) {
      acc[userId] = { messageCount: 0, totalCharacters: 0 };
    }
    acc[userId].messageCount += 1; // $group with $sum
    acc[userId].totalCharacters += m.text.length;
    return acc;
  }, {})
  .sort((a, b) => b.messageCount - a.messageCount) // $sort
  .slice(0, 10); // $limit
```

### Example 2: Average Message Length Per Model

```javascript
const avgLengths = await Message.aggregate([
  {
    $match: {
      role: 'assistant', // Only AI responses
      model: { $exists: true }
    }
  },

  {
    $group: {
      _id: '$model',
      avgLength: {
        $avg: { $strLenCP: '$text' }
      },
      count: { $sum: 1 }
    }
  },

  {
    $sort: { avgLength: -1 }
  }
]);

// Result:
[
  { _id: 'gpt-4', avgLength: 450, count: 1000 },
  { _id: 'claude-3-sonnet', avgLength: 425, count: 800 },
  { _id: 'gpt-3.5-turbo', avgLength: 320, count: 1200 }
]
```

### Common Aggregation Operators

```javascript
{
  $match: { ... },      // Filter documents (like .filter())
  $group: { ... },      // Group and aggregate (like .reduce())
  $sort: { ... },       // Sort (like .sort())
  $limit: 10,           // Limit results (like .slice(0, 10))
  $skip: 20,            // Skip documents (pagination)
  $project: { ... },    // Select/transform fields (like .map())
  $unwind: '$array',    // Flatten array field
  $lookup: { ... },     // Join with other collection
  $count: 'total'       // Count documents
}
```

---

## Indexes: Making Queries Fast

**Indexes speed up queries** (like a book's index)

### Without Index vs With Index

```javascript
// Without index: MongoDB scans ALL documents (slow)
await Message.find({ conversationId: '123' });
// Scans 1,000,000 messages → Takes 5 seconds

// With index: MongoDB jumps directly to matching documents (fast)
messageSchema.index({ conversationId: 1 });
await Message.find({ conversationId: '123' });
// Scans 50 messages → Takes 5 milliseconds
```

### Creating Indexes

```javascript
const messageSchema = new Schema({
  conversationId: {
    type: ObjectId,
    index: true // Single field index
  },

  userId: {
    type: ObjectId,
    index: true
  },

  role: String,
  text: String,
  createdAt: Date
});

// Compound index (multiple fields)
messageSchema.index({ conversationId: 1, createdAt: -1 });
// 1 = ascending, -1 = descending

// Unique index (enforce uniqueness)
userSchema.index({ email: 1 }, { unique: true });

// Text index (full-text search)
messageSchema.index({ text: 'text' });

// Sparse index (only documents with field)
userSchema.index({ lastLoginAt: 1 }, { sparse: true });
```

### When to Create Indexes

✅ **Create index for:**
- Fields used in `find()` queries
- Fields used in `sort()`
- Foreign keys (ObjectId references)
- Unique fields (email, username)

❌ **Don't index:**
- Every field (slows down writes)
- Fields rarely queried
- Fields with low selectivity (boolean)

### Index Trade-offs

**Benefits:**
- ⚡ Much faster reads

**Costs:**
- 💾 Uses disk space
- 🐌 Slower writes (index must be updated)

**Rule of thumb:** Index fields you query often, don't index everything.

### Checking Query Performance

```javascript
// Explain query execution
const explain = await Message.find({ conversationId: '123' }).explain();

console.log(explain.executionStats);
// {
//   executionTimeMillis: 5,
//   totalDocsExamined: 50,      // With index
//   nReturned: 50
// }

// vs without index:
// {
//   executionTimeMillis: 5000,
//   totalDocsExamined: 1000000, // Scanned all docs!
//   nReturned: 50
// }
```

---

## Data Modeling Best Practices

### 1. Embed vs Reference

**Embed when:**
- Data is small
- Data doesn't change often
- Data is always accessed with parent

```javascript
// Good: Embed settings (small, accessed together)
const conversationSchema = new Schema({
  title: String,
  settings: {
    model: String,
    temperature: Number
  }
});
```

**Reference when:**
- Data is large
- Data changes frequently
- Data is accessed independently
- Data is shared across documents

```javascript
// Good: Reference user (accessed separately)
const messageSchema = new Schema({
  text: String,
  userId: { type: ObjectId, ref: 'User' }
});
```

### 2. Avoid Deeply Nested Data

```javascript
// ❌ Bad: Too deeply nested
{
  user: {
    profile: {
      settings: {
        preferences: {
          theme: {
            color: 'dark'
          }
        }
      }
    }
  }
}

// ✅ Good: Flatter structure
{
  userId: ObjectId('...'),
  themeColor: 'dark'
}
```

### 3. Use Timestamps

```javascript
// Always enable timestamps
const schema = new Schema({
  // ... fields
}, {
  timestamps: true // Adds createdAt, updatedAt automatically
});
```

### 4. Don't Store What You Can Calculate

```javascript
// ❌ Bad: Store derived data
const conversationSchema = new Schema({
  messageCount: Number // Can be calculated!
});

// ✅ Good: Calculate when needed
const messageCount = await Message.countDocuments({ conversationId: '123' });

// Or cache it in Redis for performance
```

### 5. Normalize IDs in Responses

```javascript
// Convert MongoDB _id to id for frontend
messageSchema.methods.toClient = function() {
  return {
    id: this._id.toString(), // Convert ObjectId to string
    text: this.text,
    createdAt: this.createdAt
  };
};
```

---

## Migrations: Changing Schemas

**Problem:** Schema changes don't automatically update existing data

### Migration Example

**Scenario:** Add `isArchived` field to all conversations

**File:** `migrations/add-is-archived.js`

```javascript
const mongoose = require('mongoose');
const Conversation = require('../api/models/Conversation');

async function up() {
  console.log('Adding isArchived field to conversations...');

  // Update all conversations
  const result = await Conversation.updateMany(
    { isArchived: { $exists: false } }, // Only ones without field
    { $set: { isArchived: false } }
  );

  console.log(`Updated ${result.modifiedCount} conversations`);
}

async function down() {
  console.log('Removing isArchived field from conversations...');

  // Remove field
  const result = await Conversation.updateMany(
    {},
    { $unset: { isArchived: '' } }
  );

  console.log(`Updated ${result.modifiedCount} conversations`);
}

module.exports = { up, down };
```

**Run migration:**
```bash
node migrations/add-is-archived.js
```

### Schema Versioning

```javascript
const conversationSchema = new Schema({
  // ... fields

  schemaVersion: {
    type: Number,
    default: 1
  }
});

// Middleware to handle old versions
conversationSchema.post('find', function(docs) {
  docs.forEach(doc => {
    if (doc.schemaVersion === 1) {
      // Migrate old data format
      doc.settings = migrateV1Settings(doc.settings);
      doc.schemaVersion = 2;
    }
  });
});
```

---

## Performance Optimization

### 1. Use `.lean()` for Read-Only Queries

```javascript
// ❌ Slower: Returns Mongoose document (with methods, virtuals)
const messages = await Message.find({ conversationId: '123' });

// ✅ Faster: Returns plain JavaScript object
const messages = await Message.find({ conversationId: '123' }).lean();
```

**When to use `.lean()`:**
- Read-only data (not modifying)
- Sending to API response
- Large result sets

**When NOT to use `.lean()`:**
- Need to call document methods
- Need virtuals
- Will modify and save

### 2. Select Only Needed Fields

```javascript
// ❌ Bad: Fetch all fields
const users = await User.find();

// ✅ Good: Only fetch needed fields
const users = await User.find().select('name email');
// Or: .select('-password') // Exclude password
```

### 3. Limit Results

```javascript
// Always limit queries
const messages = await Message
  .find({ conversationId: '123' })
  .limit(50); // Max 50 messages
```

### 4. Pagination

```javascript
function getMessages(conversationId, page = 1, limit = 20) {
  const skip = (page - 1) * limit;

  return Message
    .find({ conversationId })
    .sort({ createdAt: -1 })
    .limit(limit)
    .skip(skip)
    .lean();
}

// Usage
const page1 = await getMessages('123', 1, 20); // Messages 1-20
const page2 = await getMessages('123', 2, 20); // Messages 21-40
```

### 5. Avoid N+1 Queries

```javascript
// ❌ Bad: N+1 queries (1 + N)
const messages = await Message.find({ conversationId: '123' });
for (const message of messages) {
  const user = await User.findById(message.userId); // N queries!
}

// ✅ Good: Single query with populate
const messages = await Message
  .find({ conversationId: '123' })
  .populate('userId', 'name email');
```

### 6. Use Aggregations for Complex Queries

```javascript
// ❌ Bad: Multiple queries
const conversations = await Conversation.find({ userId: '123' });
const messagesPerConv = await Promise.all(
  conversations.map(conv =>
    Message.countDocuments({ conversationId: conv._id })
  )
);

// ✅ Good: Single aggregation
const stats = await Message.aggregate([
  { $match: { userId: '123' } },
  { $group: { _id: '$conversationId', count: { $sum: 1 } } }
]);
```

---

## Common Patterns in LibreChat

### Pattern 1: Conversation with Messages

```javascript
// Get conversation with last 50 messages
const conversation = await Conversation.findById(conversationId);

const messages = await Message
  .find({ conversationId })
  .sort({ createdAt: 1 })
  .limit(50)
  .populate('userId', 'name avatar')
  .lean();

// Return to frontend
res.json({
  conversation: {
    id: conversation._id,
    title: conversation.title,
    model: conversation.model
  },
  messages: messages.map(m => ({
    id: m._id,
    role: m.role,
    text: m.text,
    user: m.userId,
    createdAt: m.createdAt
  }))
});
```

### Pattern 2: User's Conversations with Message Count

```javascript
const conversations = await Conversation.aggregate([
  // Match user's conversations
  { $match: { userId: new ObjectId(userId) } },

  // Lookup messages count
  {
    $lookup: {
      from: 'messages',
      localField: '_id',
      foreignField: 'conversationId',
      as: 'messages'
    }
  },

  // Add message count field
  {
    $addFields: {
      messageCount: { $size: '$messages' }
    }
  },

  // Remove messages array (don't need it)
  {
    $project: {
      messages: 0
    }
  },

  // Sort by most recent
  { $sort: { updatedAt: -1 } }
]);
```

### Pattern 3: Soft Delete (Don't Actually Delete)

```javascript
const conversationSchema = new Schema({
  // ... fields

  deleted: {
    type: Boolean,
    default: false
  },

  deletedAt: Date
});

// "Delete" conversation (soft delete)
await Conversation.findByIdAndUpdate(conversationId, {
  deleted: true,
  deletedAt: new Date()
});

// Find non-deleted conversations
const conversations = await Conversation.find({
  userId: '123',
  deleted: { $ne: true } // Not deleted
});

// Permanent delete (admin only)
await Conversation.findByIdAndDelete(conversationId);
```

### Pattern 4: Optimistic Locking (Prevent Concurrent Updates)

```javascript
const conversationSchema = new Schema({
  // ... fields

  __v: { type: Number, select: false } // Version key (automatic)
});

// Update with version check
async function updateConversation(id, updates, currentVersion) {
  const result = await Conversation.findOneAndUpdate(
    {
      _id: id,
      __v: currentVersion // Only update if version matches
    },
    {
      ...updates,
      $inc: { __v: 1 } // Increment version
    },
    { new: true }
  );

  if (!result) {
    throw new Error('Conversation was modified by another user');
  }

  return result;
}
```

---

## Summary

**Key Takeaways:**

1. **MongoDB = NoSQL Database**
   - Collections (like arrays) of documents (like objects)
   - Flexible schema, JSON-like data

2. **Mongoose = ODM (TypeScript + Validation for MongoDB)**
   - Define schemas with validation
   - Helper methods for queries
   - Middleware (hooks)

3. **Schemas = TypeScript Interfaces + Runtime Validation**
   - Define data structure
   - Validate on save
   - Custom methods and virtuals

4. **Relationships:**
   - References (ObjectId) for large/shared data
   - Embedded for small data always accessed together

5. **Queries:**
   - Basic: `find()`, `findOne()`, `findById()`
   - Advanced: Filters, sorting, pagination
   - Aggregations for complex operations

6. **Indexes:**
   - Speed up queries dramatically
   - Trade-off: Slower writes, more disk space
   - Index frequently queried fields

7. **Performance:**
   - Use `.lean()` for read-only
   - Select only needed fields
   - Avoid N+1 queries (use populate)
   - Limit and paginate results

---

**Next Steps:**

- Read [INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md) - Connect frontend to backend
- Read [HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md) - Step-by-step common tasks
- Practice queries in MongoDB Shell

---

**Back to:** [Learning Path Home](./README.md)

---

*Last updated: November 19, 2025*
*LibreChat version: v0.8.1-rc1*
