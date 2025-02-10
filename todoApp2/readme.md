# Todo App - Phase 2

## 📌 Overview
This is **Phase 2** of our Todo App, where we implement **GET, PUT, and DELETE** methods. These additions allow us to fetch all todos, retrieve a specific todo by ID, update a todo, and delete a todo.

## 📁 Folder Structure
```
todoApp/
│-- config/
│   ├── database.js
│-- controllers/
│   ├── createTodo.js
│   ├── getTodo.js
│   ├── updateTodo.js
│   ├── deleteTodo.js
│-- models/
│   ├── Todo.js
│-- routes/
│   ├── todos.js
│-- .env
│-- index.js
```

## 🛠 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST   | `/api/v1/createTodo` | Create a new todo |
| GET    | `/api/v1/getTodos` | Fetch all todos |
| GET    | `/api/v1/getTodos/:id` | Fetch a single todo by ID |
| PUT    | `/api/v1/updateTodo/:id` | Update a todo by ID |
| DELETE | `/api/v1/deleteTodo/:id` | Delete a todo by ID |

---

## 📜 API Implementation

### 1️⃣ **GET - Fetch All Todos**
**File:** `/controllers/getTodo.js`
```javascript
const Todo = require("../models/Todo");

exports.getTodo = async (req, res) => {
    try {
        const todos = await Todo.find({});
        res.status(200).json({
            success: true,
            data: todos,
            message: "Entire Todo Data is fetched",
        });
    } catch (err) {
        res.status(500).json({
            success: false,
            error: err.message,
            message: 'Server Error',
        });
    }
};
```

### 2️⃣ **GET - Fetch Todo by ID**
```javascript
exports.getTodoById = async (req, res) => {
    try {
        const id = req.params.id;
        const todo = await Todo.findById(id);
        
        if (!todo) {
            return res.status(404).json({
                success: false,
                message: "No Data Found with Given Id",
            });
        }
        res.status(200).json({
            success: true,
            data: todo,
            message: `Todo ${id} data successfully fetched`,
        });
    } catch (err) {
        res.status(500).json({
            success: false,
            error: err.message,
            message: 'Server Error',
        });
    }
};
```

### 3️⃣ **PUT - Update Todo by ID**
**File:** `/controllers/updateTodo.js`
```javascript
exports.updateTodo = async (req, res) => {
    try {
        const { id } = req.params;
        const { title, description } = req.body;

        const todo = await Todo.findByIdAndUpdate(
            id,
            { title, description, updatedAt: Date.now() },
            { new: true }
        );

        res.status(200).json({
            success: true,
            data: todo,
            message: "Updated Successfully",
        });
    } catch (err) {
        res.status(500).json({
            success: false,
            error: err.message,
            message: 'Server Error',
        });
    }
};
```

### 4️⃣ **DELETE - Remove Todo by ID**
**File:** `/controllers/deleteTodo.js`
```javascript
exports.deleteTodo = async (req, res) => {
    try {
        const { id } = req.params;
        await Todo.findByIdAndDelete(id);
        res.json({
            success: true,
            message: "Todo DELETED",
        });
    } catch (err) {
        res.status(500).json({
            success: false,
            error: err.message,
            message: 'Server Error',
        });
    }
};
```

---

## 📌 **Routes Configuration**
**File:** `/routes/todos.js`
```javascript
const express = require("express");
const router = express.Router();

const { createTodo } = require("../controllers/createTodo");
const { getTodo, getTodoById } = require("../controllers/getTodo");
const { updateTodo } = require("../controllers/updateTodo");
const { deleteTodo } = require("../controllers/deleteTodo");

// Define API routes
router.post("/createTodo", createTodo);
router.get("/getTodos", getTodo);
router.get("/getTodos/:id", getTodoById);
router.put("/updateTodo/:id", updateTodo);
router.delete("/deleteTodo/:id", deleteTodo);

module.exports = router;
```

---

## 📚 What I Learned Today
✅ **GET Method** - Fetch all or a specific todo using `find()` and `findById()`.
✅ **PUT Method** - Update a todo using `findByIdAndUpdate()`.
✅ **DELETE Method** - Remove a todo using `findByIdAndDelete()`.
✅ **Error Handling** - Implementing error messages for better debugging.
✅ **Modular Code Structure** - Separating controllers, routes, and models for scalability.
---

This concludes **Phase 2** of our **Todo App**! 🎯 Let me know if you need any improvements or explanations! 😃

