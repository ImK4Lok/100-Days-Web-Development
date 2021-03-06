# Image Picker
> Implementing an Image Picker with `HTML`, handle the uploaded image and storage with `node/multer` and preview the selected image with `JavaScript`.

## Create an Image Picker
> The `<input>` element has an attribute `type="file"` to create an file picker, and we can select which file type we accept with attribute `accept="image/jpg"`
```html
<main id="new-user">
    <form action="/profiles" method="POST">
      <div class="form-control">
        <label for="username">Username</label>
        <input type="text" id="username" name="username" required>
      </div>
      <div class="form-control">
        <label for="image">User image</label>
        <input type="file" id="image" name="image" accept="image/jpg,image/png" required />
      </div>
      <button class="btn">Save User</button>
    </form>
</main>
```

### Form attribute - `enctype`
> This attribute specifies how the form-data shoule be encoded when submitting to the server, it only works when `method="POST"`.

By default, the value is `enctype="application/x-www-form-urlencoded"`, which will encode all characters before sent. However, if we want to upload a file through form, the value `enctype="multipart/form-data"` is necessary.
```html
<form action="" method="POST" enctype="multipart/form-data">
```

---

## Handling the Upload file
> To manage parsing and storing of the uploaded file from the form data (POST request) by using `multer`. A node.js middleware for handling multipart/form-data.
1. Install it and requires it
```js
// console
> npm install --save multer

// app.js
const multer = require('multer');
```
2. Config the options
```js
// basic use - the rest of it will be by default
const upload = multer({dest: 'uploads/'});

// customed use - more options
// .diskStorage creates the disk storage object for full control on storing files to disk
const storageConfig = multer.diskStorage({
    destination: function(req, file, cb) {
        // first param: potential error might occur
        // second param: path to store the files
        cb(null, 'images');
    },
    // this property can help name the uploaded file
    filename: ,function(req, file, cb) {
        // make the file path like this -> "> "1657839412660-myAvatar.jpg"
        cb(null, Date.now() + '-' + file.originalname);
    }
});
```

3. Use the middleware
> The router `.get` or `.post` get unlimited middleware function to call from left to right in order, therefore, the `upload.single()` can parse the file data.
```js
// multer will store the file to the pre-set path by itself
router.post('/profiles', upload.single('image'), function(req, res) {
    const uploadedImageFile = req.file;
    const userData = req.body;
    
    // insert the data and image path into the database
    await db.getDB().collection('users').insertOne({
        name: userData.username,
        imagePath: uploadedImageFile.path
    });
    
    res.redirect('/');
});
```

---

## Preview the Uploaded file
> To preview the uploaded file immediately, we need javascript to make it smoother without reloading the page in order the refresh the webpage.
1. Lets create a script to handle the preview function
```js
// in the .ejs or .html
<script src="scripts/file-preview.js" defer></script>
```
2. Add event listen to the image picker
```js
const filePickerElement = document.getElementById('image');
const imagePreviewElement = document.getElementById('image-preview');
// the 'change' event will be fired after the value of the picker is changed
filePickerElement.addEventListener('change', showPreview);
```
3. Create the show preview function
```js
// The files will be stored inside the filePickerElement after picker the file
function showPreview() {
    const files = filePickerElement.files;
    // checkt if files is null
    if(!files || files.length === 0) {
        imagePreviewElement.style.display = "none";
        return;
    }
    // appoarch if we know there's only one file to pick
    const pickedFile = files[0];
    // .createObjectURL returns a path from the browser' side machine, that're picked by the user
    imagePreviewElement.src = URL.createObjectURL(pickedFile);
    imagePreviewElement.style.display = "block";
}

```

---
