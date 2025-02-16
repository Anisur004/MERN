# Authentication & Authorization - Learning Notes

## What I Learned Today 📝

Today, I learned about:
- **Authentication & Authorization**
- **JWT (JSON Web Token)**
- **Cookies for session management**
- **bcrypt Hash Function** for securing passwords
- Implementing **Signup Feature** with proper validation & error handling

---

## 🔐 Authentication vs Authorization

| Feature | Description |
|---------|-------------|
| **Authentication** | Verifies **who you are** (e.g., login with email/password) |
| **Authorization** | Verifies **what you can access** (e.g., admin vs regular user) |

---

## 🛠️ JSON Web Token (JWT)
**JWT (JSON Web Token)** is used for securely transmitting user information between client and server.
- **Header** → Contains metadata like algorithm used
- **Payload** → User data (e.g., `id`, `email`, `role`)
- **Signature** → Ensures token integrity (hashed using a secret key)

---

## 🍪 Cookies for Authentication
- **Cookies** are small pieces of data stored in the browser.
- Used to **persist authentication** (e.g., keeping a user logged in)
- Can store JWT **as a HTTP-only cookie** (prevents client-side access)

---

## 🔑 bcrypt Hash Function
**bcrypt** is used to securely store user passwords.
- Converts plain text passwords into **hashed values**
- Prevents storing raw passwords in the database
- Uses **salting** (random data) to enhance security

---

## ✨ Signup Feature Implementation

### **Step-by-Step Process**
1. **Extract user data** from request body
2. **Check if user already exists** in the database
3. **If user exists**, return an error response
4. **Hash the password** using bcrypt before storing
5. **Insert the new user into the database**
6. **Return success response**, or error if something fails

---

---
## Signup Feature Implementation
Below is the **signup handler** code, broken into sections with explanations.

### **Code:**
```javascript
const bcrypt = require("bcrypt");
const User = require("../models/User");
const jwt = require("jsonwebtoken");
require("dotenv").config();

// Signup route handler
exports.signup = async (req, res) => {
    try {
        // Step 1: Get data from request body
        const { name, email, password, role } = req.body;

        // Step 2: Check if the user already exists
        const existingUser = await User.findOne({ email });
        if (existingUser) {
            return res.status(400).json({
                success: false,
                message: "User already exists",
            });
        }

        // Step 3: Hash the password for security
        let hashedPassword;
        try {
            hashedPassword = await bcrypt.hash(password, 10);
        } catch (err) {
            return res.status(500).json({
                success: false,
                message: "Error in hashing password",
            });
        }

        // Step 4: Store user data in the database
        const user = await User.create({
            name,
            email,
            password: hashedPassword,
            role
        });

        // Step 5: Return success response
        return res.status(200).json({
            success: true,
            message: "User created successfully",
        });

    } catch (error) {
        console.error(error);
        return res.status(500).json({
            success: false,
            message: "User cannot be registered, please try again later",
        });
    }
};
```

### **Code Breakdown:**
#### 🟢 **Step 1: Extract Data from Request Body**
```javascript
const { name, email, password, role } = req.body;
```
- The user submits a **name, email, password, and role**.
- These values are extracted from the request body.

#### 🟢 **Step 2: Check if the User Already Exists**
```javascript
const existingUser = await User.findOne({ email });
if (existingUser) {
    return res.status(400).json({
        success: false,
        message: "User already exists",
    });
}
```
- Checks the database to see if an account with the **same email** exists.
- If the user **already exists**, an error response is returned.

#### 🟢 **Step 3: Hash the Password**
```javascript
let hashedPassword;
try {
    hashedPassword = await bcrypt.hash(password, 10);
} catch (err) {
    return res.status(500).json({
        success: false,
        message: "Error in hashing password",
    });
}
```
- Uses **bcrypt** to hash the password before storing it.
- The second argument `10` is the **salt rounds** (higher value = more security but slower performance).
- If hashing fails, an error response is returned.

#### 🟢 **Step 4: Store the User in the Database**
```javascript
const user = await User.create({
    name,
    email,
    password: hashedPassword,
    role
});
```
- Stores the new user **with hashed password** in the database.

#### 🟢 **Step 5: Return Success Response**
```javascript
return res.status(200).json({
    success: true,
    message: "User created successfully",
});
```
- Sends a **success response** to the client if everything works correctly.


## Authentication - Part 2: Implementing Login

Today, I worked on implementing the **login** functionality in my authentication system. This process includes:

1. **Fetching email and password** from the request body.
2. **Validating email and password** (checking if fields are empty).
3. **Checking if the user is registered** in the database.
4. **Comparing passwords** using bcrypt.
5. **Creating a JWT token** for authentication.
6. **Storing the token in cookies** for future authentication.

---

## Login API Code

```javascript
exports.login = async (req, res) => {
    try {
        // Step 1: Fetch email and password from request body
        const { email, password } = req.body;

        // Step 2: Validate email and password
        if (!email || !password) {
            return res.status(400).json({
                success: false,
                message: 'Please fill all the details carefully',
            });
        }

        // Step 3: Check if the user is registered
        let user = await User.findOne({ email });
        if (!user) {
            return res.status(401).json({
                success: false,
                message: 'User is not registered',
            });
        }

        // Step 4: Create a payload for JWT token
        const payload = {
            email: user.email,
            id: user._id,
            role: user.role,
        };

        // Step 5: Verify password and generate a JWT token
        if (await bcrypt.compare(password, user.password)) {
            // Password match
            let token = jwt.sign(payload, process.env.JWT_SECRET, {
                expiresIn: '2h',
            });

            // Step 6: Convert user to a plain object and remove password
            user = user.toObject();
            user.token = token;
            user.password = undefined;

            // Step 7: Set token in HTTP cookie
            const options = {
                expires: new Date(Date.now() + 3 * 24 * 60 * 60 * 1000),
                httpOnly: true,
            };

            res.cookie('authToken', token, options).status(200).json({
                success: true,
                token,
                user,
                message: 'User logged in successfully',
            });
        } else {
            // Step 8: Handle incorrect password
            return res.status(403).json({
                success: false,
                message: 'Password incorrect',
            });
        }
    } catch (error) {
        console.error(error);
        return res.status(500).json({
            success: false,
            message: 'Login failure',
        });
    }
};
```

---

## Code Breakdown and Explanation

### **1. Fetch Email and Password from Request Body**
```javascript
const { email, password } = req.body;
```
- Extracts `email` and `password` from the incoming request.

### **2. Validate Email and Password**
```javascript
if (!email || !password) {
    return res.status(400).json({
        success: false,
        message: 'Please fill all the details carefully',
    });
}
```
- Ensures that the user has provided both `email` and `password`.
- If any field is missing, returns a `400 Bad Request` error.

### **3. Check if User is Registered**
```javascript
let user = await User.findOne({ email });
if (!user) {
    return res.status(401).json({
        success: false,
        message: 'User is not registered',
    });
}
```
- Searches for the user in the database using `email`.
- If the user is not found, returns a `401 Unauthorized` error.

### **4. Create Payload for JWT Token**
```javascript
const payload = {
    email: user.email,
    id: user._id,
    role: user.role,
};
```
- Creates a `payload` object that will be stored inside the JWT token.

### **5. Compare Passwords using bcrypt**
```javascript
if (await bcrypt.compare(password, user.password)) {
```
- Compares the entered password with the **hashed password** stored in the database.
- If the passwords match, authentication is successful.

### **6. Generate JWT Token**
```javascript
let token = jwt.sign(payload, process.env.JWT_SECRET, {
    expiresIn: '2h',
});
```
- Creates a **JWT token** using the `jsonwebtoken` library.
- The token expires in **2 hours**.

### **7. Convert User to Plain Object and Remove Password**
```javascript
user = user.toObject();
user.token = token;
user.password = undefined;
```
- Converts the user object to a **plain JavaScript object**.
- Removes the **password** to prevent it from being exposed.

### **8. Set Token in Cookies**
```javascript
const options = {
    expires: new Date(Date.now() + 3 * 24 * 60 * 60 * 1000),
    httpOnly: true,
};

res.cookie('authToken', token, options).status(200).json({
    success: true,
    token,
    user,
    message: 'User logged in successfully',
});
```
- Stores the JWT token in an **HTTP-only cookie**.
- The cookie expires in **3 days**.
- Sends a `200 OK` response with user details.

### **9. Handle Incorrect Password**
```javascript
return res.status(403).json({
    success: false,
    message: 'Password incorrect',
});
```
- If the password is incorrect, returns a `403 Forbidden` error.

### **10. Catch Errors**
```javascript
catch (error) {
    console.error(error);
    return res.status(500).json({
        success: false,
        message: 'Login failure',
    });
}
```
- If any unexpected error occurs, returns a `500 Internal Server Error`.

---

## **Summary of Login Flow**
1. **Extract email & password** from request body.
2. **Validate input** (check if fields are empty).
3. **Check if user exists** in the database.
4. **Compare password** using bcrypt.
5. **Generate a JWT token** if password is correct.
6. **Store the token in an HTTP cookie**.
7. **Send response with token and user data** (excluding password).
8. **Return appropriate error messages** if authentication fails.

---



## Today I Learned: Authentication & Authorization Middleware

In the third part of authentication, I learned how middleware works in authentication and authorization.

### What is Middleware?
Middleware functions are functions that execute during the request-response cycle and have access to the request (`req`), response (`res`), and the `next()` function. Middleware is commonly used for authentication, logging, validation, and error handling.

## Protected Routes Implementation

We created three protected routes using middleware:

```javascript
// Testing a protected route with a single middleware
router.get("/test", auth, (req,res) =>{
    res.json({
        success:true,
        message:'Welcome to the Protected route for TESTS',
    });
});

// Protected Route for Students
router.get("/student", auth, isStudent, (req,res) => {
    res.json({
        success:true,
        message:'Welcome to the Protected route for Students',
    });
} );

// Protected Route for Admin
router.get("/admin", auth, isAdmin, (req,res) => {
    res.json({
        success:true,
        message:'Welcome to the Protected route for Admin',
    });
});
```

### Middleware for Authentication and Authorization
We created an `auth.js` file inside the `middleware` folder to handle authentication and role-based access control.

### 1. Authentication Middleware (`auth`)
This middleware verifies the JWT token provided by the user and ensures that the request is coming from an authenticated user.

```javascript
const jwt = require("jsonwebtoken");
require("dotenv").config();

exports.auth = (req, res, next) => {
    try {
        // Extract JWT token from request body (other ways can also be used)
        const token = req.body.token;
        
        if (!token) {
            return res.status(401).json({
                success: false,
                message: 'Token Missing',
            });
        }

        // Verify the token
        try {
            const payload = jwt.verify(token, process.env.JWT_SECRET);
            console.log(payload);
            // Attach user details to request object
            req.user = payload;
        } catch (error) {
            return res.status(401).json({
                success: false,
                message: 'Token is invalid',
            });
        }
        next(); // Pass control to next middleware
    } catch (error) {
        return res.status(401).json({
            success: false,
            message: 'Something went wrong while verifying the token',
        });
    }
};
```

### 2. Authorization Middleware for Students (`isStudent`)
This middleware checks if the authenticated user has the `Student` role.

```javascript
exports.isStudent = (req, res, next) => {
    try {
        if (req.user.role !== "Student") {
            return res.status(401).json({
                success: false,
                message: 'This is a protected route for students',
            });
        }
        next(); // Pass control to next middleware
    } catch (error) {
        return res.status(500).json({
            success: false,
            message: 'User Role is not matching',
        });
    }
};
```

### 3. Authorization Middleware for Admins (`isAdmin`)
This middleware ensures that only users with the `Admin` role can access certain routes.

```javascript
exports.isAdmin = (req, res, next) => {
    try {
        if (req.user.role !== "Admin") {
            return res.status(401).json({
                success: false,
                message: 'This is a protected route for admin',
            });
        }
        next(); // Pass control to next middleware
    } catch (error) {
        return res.status(500).json({
            success: false,
            message: 'User Role is not matching',
        });
    }
};
```

## Explanation - Breaking the Code into Steps

### 1. Setting Up Protected Routes
- The `/test` route is protected using only the `auth` middleware.
- The `/student` route is protected using `auth` and `isStudent` middleware.
- The `/admin` route is protected using `auth` and `isAdmin` middleware.

### 2. Authentication Middleware (`auth`)
- Extracts the token from the request body.
- Checks if the token exists; if not, returns a `401 Unauthorized` response.
- Verifies the token using `jwt.verify()`.
- If the token is valid, attaches the decoded user information to `req.user`.
- Calls `next()` to proceed to the next middleware or route handler.

### 3. Authorization Middleware (`isStudent` and `isAdmin`)
- Checks if `req.user.role` matches the required role (either `Student` or `Admin`).
- If the role does not match, returns a `401 Unauthorized` response.
- If the role matches, calls `next()` to proceed to the next middleware or route handler.

## Summary
- Middleware functions are used to handle authentication and authorization.
- `auth` middleware verifies the JWT token and extracts user details.
- `isStudent` and `isAdmin` middleware restrict access to specific user roles.
- Protected routes ensure only authorized users can access sensitive data.

# Learning Notes - Authentication (Part 4: Cookie Parsing)

## Today’s Learning: Cookie Parsing in Authentication

### What is Cookie Parsing?
Cookies are small pieces of data stored on the client-side (browser) that help in maintaining user sessions. In authentication, cookies are commonly used to store JWT (JSON Web Tokens) to manage user sessions securely.

### Two Ways of Cookie Parsing:
There are two common ways to parse cookies in a Node.js/Express application:

#### 1. Using `req.headers.cookie`
   - The `req.headers.cookie` property retrieves the raw cookie string from the request headers.
   - This method requires manual parsing to extract values.

   **Example:**
   ```javascript
   app.get("/read-cookie", (req, res) => {
       const rawCookies = req.headers.cookie;
       console.log("Raw Cookies:", rawCookies);
       res.send("Cookies received");
   });
   ```

#### 2. Using `cookie-parser` Middleware (Recommended)
   - `cookie-parser` is an Express middleware that automatically parses cookies into an object.
   - It allows easy access to cookies via `req.cookies`.

   **Installation:**
   ```sh
   npm install cookie-parser
   ```

   **Usage in Express:**
   ```javascript
   const express = require("express");
   const cookieParser = require("cookie-parser");

   const app = express();
   app.use(cookieParser()); // Middleware to parse cookies

   app.get("/read-cookie", (req, res) => {
       console.log("Cookies:", req.cookies);
       res.send("Cookies received");
   });
   ```

### Setting Cookies with Expiration Time:
To set a cookie with an expiration time, we can use the `res.cookie()` method.

**Example:**
```javascript
app.get("/set-cookie", (req, res) => {
    res.cookie("authToken", "secureTokenValue", {
        maxAge: 2 * 60 * 60 * 1000, // 2 hours
        httpOnly: true, // Prevents client-side JavaScript from accessing the cookie
        secure: true,  // Ensures the cookie is sent over HTTPS
    });
    res.send("Cookie has been set");
});
```

### Time Period of Cookies:
- `maxAge`: Defines cookie lifetime in milliseconds.
- `expires`: Defines an exact date when the cookie should expire.
- `session cookie`: If no expiry is set, the cookie is deleted when the browser is closed.

### Summary:
1. Cookies help manage authentication sessions.
2. There are two ways to parse cookies: `req.headers.cookie` (manual parsing) and `cookie-parser` (recommended middleware).
3. Cookies can be set with an expiration time using `maxAge` or `expires`.
4. `httpOnly` and `secure` flags improve security by preventing JavaScript access and ensuring cookies are sent only over HTTPS.

---
This completes today’s learning on **Cookie Parsing in Authentication**. 🚀


