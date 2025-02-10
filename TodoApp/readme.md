# 📌 Brief Notes on Today's Learning

## 1️⃣ Setting Up a Node.js Project

### **Folder & File Structure:**
```
project-root/
│-- config/
│   └── database.js
│-- controllers/
│   └── reateTodo.js
│-- models/
│   └── todo.js
│-- routes/
│   └── todos.js
│-- .env
│-- index.js
```

### **Folder & File Purpose:**
- **config/** → Contains database connection settings.
- **controllers/** → Houses route handlers (business logic for APIs).
- **models/** → Defines the database schema using Mongoose.
- **routes/** → Defines API endpoints and maps them to controllers.
- **.env** → Stores environment variables like `PORT` and `DATABASE_URL`.
- **index.js** → The main entry point of the server.

---

## 2️⃣ Index.js - Setting Up Express Server

### **What Happens in index.js?**
- Loads environment variables from `.env`.
- Initializes Express and middleware to parse JSON requests.
- Imports and mounts routes from `routes/todos.js`.
- Connects to MongoDB using `dbConnect()`.
- Defines a default homepage route.
- Starts the server on the specified `PORT`.

### **Code:**
```js
const express = require("express");
const app = express();

require("dotenv").config();
const PORT = process.env.PORT || 4000;

app.use(express.json());

const todoRoutes = require("./routes/todos");
app.use("/api/v1", todoRoutes);

const dbConnect = require("./config/database");
dbConnect();

app.get("/", (req, res) => {
    res.send(`<h1> This is HOMEPAGE baby</h1>`);
});

app.listen(PORT, () => {
    console.log(`Server started successfully at ${PORT}`);
});
```

---

## 3️⃣ Database Connection - config/database.js

### **Purpose:**
Handles MongoDB connection using Mongoose.

### **Code:**
```js
const mongoose = require("mongoose");
require("dotenv").config();

const dbConnect = () => {
    mongoose.connect(process.env.DATABASE_URL, {
        useNewUrlParser: true,
        useUnifiedTopology: true,
    })
    .then(() => console.log("DB ka Connection is Successful"))
    .catch((error) => {
        console.log("Issue in DB Connection");
        console.error(error.message);
        process.exit(1); // Exit process on failure
    });
};

module.exports = dbConnect;
```

---

## 4️⃣ Defining Mongoose Schema - models/todo.js

### **Purpose:**
Defines the schema for storing Todo items in MongoDB.

### **Code:**
```js
const mongoose = require("mongoose");

const todoSchema = new mongoose.Schema({
    title: {
        type: String,
        required: true,
        maxLength: 50,
    },
    description: {
        type: String,
        required: true,
        maxLength: 50,
    },
    createdAt: {
        type: Date,
        required: true,
        default: Date.now,
    },
    updatedAt: {
        type: Date,
        required: true,
        default: Date.now,
    }
});

module.exports = mongoose.model("Todo", todoSchema);
```

---

## 5️⃣ Controller Logic - controllers/todo.js

### **Purpose:**
Defines the logic for handling the creation of Todo entries.

### **Code:**
```js
const Todo = require("../models/todo");

exports.createTodo = async (req, res) => {
    try {
        const { title, description } = req.body;
        const response = await Todo.create({ title, description });
        res.status(200).json({
            success: true,
            data: response,
            message: "Entry Created Successfully"
        });
    } catch (err) {
        console.error(err);
        res.status(500).json({
            success: false,
            data: "Internal Server Error",
            message: err.message,
        });
    }
};
```

---

## 6️⃣ Setting Up Routes - routes/todos.js

### **Purpose:**
Defines API endpoints and maps them to controllers.

### **Code:**
```js
const express = require("express");
const router = express.Router();

const { createTodo } = require("../controllers/todo");

router.post("/createTodo", createTodo);

module.exports = router;
```

---

## 7️⃣ Environment Variables - .env

### **Purpose:**
Stores configuration variables like PORT and DATABASE_URL.

### **Content:**
```
PORT=3000
DATABASE_URL=mongodb://127.0.0.1:27017/ToDo
```

---

# 📚 What You Learned Today

## ✅ Phase 1 of ToDo App:
- **Folder Structure**: Organized MVC pattern.
- **Express Server**: Set up `index.js` with routes and middleware.
- **Database Connection**: Connected MongoDB using `mongoose`.
- **Schema Definition**: Created a Mongoose schema for Todos.
- **Controller Logic**: Implemented API for creating Todos.
- **Routing**: Defined API endpoints in `routes/todos.js`.
- **Environment Variables**: Used `.env` for configuration.

---

# 🌟 Next Steps (Phase 2)

🔹 Implement **Read (GET) Todos API**.
🔹 Implement **Update & Delete** operations.
🔹 Add **Validation & Error Handling**.
🔹 Test APIs using **Postman** or **Thunder Client**.

Would you like a hands-on challenge to practice this? 🚀

