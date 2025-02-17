


# 📌 Today's Learning  

## 1️⃣ Image & Video Upload in Cloudinary  
- **Cloudinary** is used to store images and videos in the cloud.  
- Steps to upload:  
  1. Install `cloudinary` package.  
  2. Configure Cloudinary with API keys.  
  3. Use Cloudinary's upload method to store files.  

- **Image Compression Before Upload**:  
  - Compress images using `multer` or other libraries before uploading to save space and bandwidth.  

---

## 2️⃣ Pre & Post Middleware in Mongoose  
- **Pre Middleware**: Executes **before** an operation (e.g., saving a document).  
- **Post Middleware**: Executes **after** an operation (e.g., once a document is saved).  

🔹 **Example Use Cases**:  
- **Pre Middleware**: Hashing passwords before saving a user.  
- **Post Middleware**: Logging an event after saving a record.  

---

## 3️⃣ Sending Emails with NodeMailer  
- **NodeMailer** is a package used to send emails from Node.js applications.  
- Steps to send an email:  
  1. Install `nodemailer`.  
  2. Create a transporter with SMTP service (Gmail, Mailtrap, etc.).  
  3. Define email content and send it.  

🔹 **Common Use Cases**:  
- Sending OTPs  
- Account verification  
- Password reset links  

---
## **Local File Upload**  

### **📌 `localFileUpload` Function**  
This function handles file uploads to the local server.  

### **📌 Steps:**  
1. **Fetch File from Request**  
   ```js
   const file = req.files.file;
   ```
   - Retrieves the file from the request.  

2. **Generate a File Path**  
   ```js
   let path = __dirname + "/files/" + Date.now() + `.${file.name.split('.')[1]}`;
   ```
   - Creates a unique path using `Date.now()`.  

3. **Move File to the Server**  
   ```js
   file.mv(path , (err) => { console.log(err); });
   ```
   - Moves the file to the generated path.  

4. **Send Success Response**  
   ```js
   res.json({
       success:true,
       message:'Local File Uploaded Successfully',
   });
   ```
   - Sends a success message if the file is uploaded.  

---

## *Uploading to Cloudinary**  

### **📌 `uploadFileToCloudinary` Function**  
This function uploads a file to Cloudinary.  

### **📌 Steps:**  
1. **Define Upload Options**  
   ```js
   const options = { folder };
   if(quality) options.quality = quality;
   options.resource_type = "auto";
   ```
   - Sets the folder and quality (optional) for the upload.  

2. **Upload File to Cloudinary**  
   ```js
   return await cloudinary.uploader.upload(file.tempFilePath, options);
   ```
   - Calls Cloudinary’s API to upload the file.  

---

## **Image Upload to Cloudinary**  

### **📌 `imageUpload` Function**  
Handles image upload with validation.  

### **📌 Steps:**  
1. **Fetch Data from Request**  
   ```js
   const { name, tags, email } = req.body;
   const file = req.files.imageFile;
   ```
   - Retrieves metadata and the image file.  

2. **Validate File Type**  
   ```js
   const supportedTypes = ["jpg", "jpeg", "png"];
   const fileType = file.name.split('.')[1].toLowerCase();
   if(!isFileTypeSupported(fileType, supportedTypes)) {
       return res.status(400).json({ success:false, message:'File format not supported' });
   }
   ```
   - Ensures only **jpg, jpeg, and png** files are allowed.  

3. **Upload File to Cloudinary**  
   ```js
   const response = await uploadFileToCloudinary(file, "Codehelp");
   ```
   - Uploads the image to the **"Codehelp"** folder in Cloudinary.  

4. **Save Entry to Database**  
   ```js
   const fileData = await File.create({
       name, tags, email, imageUrl: response.secure_url,
   });
   ```
   - Stores file metadata in the database.  

---

## **Video Upload to Cloudinary**  

### **📌 `videoUpload` Function**  
Handles video uploads with validation.  

### **📌 Steps:**  
1. **Validate File Type**  
   ```js
   const supportedTypes = ["mp4", "mov"];
   ```
   - Allows only **mp4 and mov** formats.  

2. **Upload Video to Cloudinary**  
   ```js
   const response = await uploadFileToCloudinary(file, "Codehelp");
   ```
   - Uploads the video file to Cloudinary.  

3. **Save Entry to Database**  
   ```js
   const fileData = await File.create({
       name, tags, email, imageUrl: response.secure_url,
   });
   ```
   - Stores video metadata in the database.  

---

## **Image Compression Before Upload**  

### **📌 `imageSizeReducer` Function**  
Handles image compression before uploading to Cloudinary.  

### **📌 Steps:**  
1. **Set Quality Parameter**  
   ```js
   const response = await uploadFileToCloudinary(file, "Codehelp", 90);
   ```
   - Reduces image quality to **90%** before uploading.  

2. **Save Compressed Image to Database**  
   ```js
   const fileData = await File.create({
       name, tags, email, imageUrl: response.secure_url,
   });
   ```
   - Stores compressed image metadata.  

---

---

# 

## **Post Middleware & NodeMailer for Email Notifications**  
Today, I learned how to use **post middleware** in Mongoose to send email notifications after saving a file in the database using **NodeMailer**.  

---

## **1️⃣ What is Post Middleware?**  
- `post` middleware in Mongoose runs **after** a document is saved in the database.  
- It allows us to execute additional logic **after saving data**, such as sending emails or logging activities.  

---

## **2️⃣ Understanding the Code**  

### **📌 `fileSchema.post("save", async function(doc) {...})`**  
- This middleware executes **after** a file is successfully saved in MongoDB.  
- It sends an email notification using **NodeMailer**.  

---

## **3️⃣ Breaking Down the Code**  

### **Step 1: Define the Post Middleware**  
```js
fileSchema.post("save", async function(doc) {
```
- This middleware is triggered whenever a document is saved in the **File** collection.  
- The `doc` parameter represents the saved document.  

---

### **Step 2: Log the Document (Optional)**  
```js
console.log("DOC", doc);
```
- Prints the saved document details in the console.  

---

### **Step 3: Create an Email Transporter**  
```js
let transporter = nodemailer.createTransport({
    host: process.env.MAIL_HOST,
    auth: {
        user: process.env.MAIL_USER,
        pass: process.env.MAIL_PASS,
    },
});
```
- **Creates a transporter** using `nodemailer.createTransport()`.  
- Uses **environment variables** to store email credentials (`MAIL_HOST`, `MAIL_USER`, `MAIL_PASS`).  
- This helps keep credentials secure.  

---

### **Step 4: Send an Email Notification**  
```js
let info = await transporter.sendMail({
    from: `CodeHelp - by Babbar`,
    to: doc.email,
    subject: "New File Uploaded on Cloudinary",
    html: `<h2>Hello Jee</h2> <p>File Uploaded View here: <a href="${doc.imageUrl}">${doc.imageUrl}</a> </p>`,
});
```
- **`to: doc.email`** → Sends an email to the user who uploaded the file.  
- **`subject: "New File Uploaded on Cloudinary"`** → Email subject line.  
- **`html: ...`** → Email content containing a clickable link to view the uploaded file.  

---

### **Step 5: Log the Email Info (Optional)**  
```js
console.log("INFO", info);
```
- Prints email details (e.g., success or failure).  

---

### **Step 6: Handle Errors**  
```js
catch(error) {
    console.error(error);
}
```
- Catches and logs any errors in sending the email.  

---


---
