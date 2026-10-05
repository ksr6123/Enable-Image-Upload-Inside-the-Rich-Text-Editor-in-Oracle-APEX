# 🖼️ How to Enable Image Upload Inside the Rich Text Editor in Oracle APEX

## 📌 Introduction

The **Rich Text Editor (RTE)** in Oracle APEX is useful for creating formatted content such as documentation, announcements, knowledge-base articles, comments, and descriptions.

However, in some use cases, users may also need to **insert or upload images directly inside the Rich Text Editor**.

In this article, we will see how to enable image upload functionality in the Oracle APEX Rich Text Editor using **JavaScript and the CKEditor upload adapter**.

The solution converts the selected image into a **Base64 Data URL** and inserts it directly into the editor content.

---

## 🛠️ Technologies Used

* Oracle APEX
* Oracle APEX Rich Text Editor
* JavaScript
* CKEditor FileRepository / Upload Adapter
* Base64 Data URL

---

## 💼 Business Requirement

Consider an application where users need to create formatted content containing:

* Text formatting
* Headings
* Lists
* Links
* Images
* Documentation content
* Knowledge-base articles
* Announcements

By default, the Rich Text Editor may not provide the required image-upload behavior for your use case.

Instead of asking users to upload images separately and then provide an image URL, we can enable an **Upload Image** option directly inside the Rich Text Editor.

The overall flow is:

```text
User selects image
       ↓
Rich Text Editor
       ↓
JavaScript Upload Adapter
       ↓
FileReader API
       ↓
Convert image to Base64
       ↓
Insert image into editor
```

---

# 🚀 Implementation

## Step 1: Create a Rich Text Editor Item

Open your Oracle APEX application and navigate to:

**Page Designer → Create Page Item**

Create a new page item and set its type to:

**Rich Text Editor**

For example:

```text
Item Name: P1_DESCRIPTION
Type: Rich Text Editor
```

Configure the item according to your application requirements.

---

## Step 2: Open Initialization JavaScript Function

Select the Rich Text Editor item.

Go to:

**Page Designer → Item → Advanced → Initialization JavaScript Function**

Add the following JavaScript code:

```javascript
function(options) {
    options.editorOptions.extraPlugins =
        options.editorOptions.extraPlugins || [];

    options.editorOptions.extraPlugins.push(
        function MyCustomUploadAdapterPlugin(editor) {

            editor.plugins.get('FileRepository').createUploadAdapter =
                (loader) => {

                    return {
                        upload() {

                            return loader.file.then(file => {

                                return new Promise((resolve, reject) => {

                                    const reader = new FileReader();

                                    reader.onload = () => {

                                        const base64 = reader.result;

                                        resolve({
                                            default: base64
                                        });

                                    };

                                    reader.onerror = reject;

                                    reader.readAsDataURL(file);
                                });
                            });
                        },

                        abort() {
                            // Upload cancellation logic can be added here
                        }
                    };
                };
        }
    );

    // Get the existing toolbar configuration
    let tb = options.editorOptions.toolbar || [];

    // Add Upload Image option if it is not already available
    if (!tb.includes('uploadImage')) {
        tb.push('uploadImage');
    }

    options.editorOptions.toolbar = tb;

    return options;
}
```

---

# 🔍 How Does the Code Work?

Let's understand the important parts of the JavaScript.

### 1. Enable the Custom Upload Adapter

```javascript
options.editorOptions.extraPlugins =
    options.editorOptions.extraPlugins || [];
```

This initializes the `extraPlugins` array if it has not already been configured.

---

### 2. Create the Upload Adapter

The following code accesses CKEditor's `FileRepository`:

```javascript
editor.plugins.get('FileRepository').createUploadAdapter =
    (loader) => {
```

This allows us to define our own upload behavior.

Instead of sending the image to an external server, we process the selected file directly in the browser.

---

### 3. Read the Selected Image

The selected file is obtained using:

```javascript
return loader.file.then(file => {
```

Then we create a JavaScript `FileReader`:

```javascript
const reader = new FileReader();
```

---

### 4. Convert the Image to Base64

The important part is:

```javascript
reader.readAsDataURL(file);
```

The image is converted into a Data URL such as:

```text
data:image/png;base64,iVBORw0KGgoAAAANSUhEUg...
```

The generated Base64 value is then returned to the editor:

```javascript
resolve({
    default: base64
});
```

The Rich Text Editor can then display the image.

---

## Step 3: Add the Upload Image Button

The following section checks whether `uploadImage` already exists in the toolbar:

```javascript
let tb = options.editorOptions.toolbar || [];

if (!tb.includes('uploadImage')) {
    tb.push('uploadImage');
}

options.editorOptions.toolbar = tb;
```

If the button is not available, it is added automatically.

This gives the user an **Upload Image** option directly inside the Rich Text Editor.

---

# ▶️ Step 4: Save and Run the Page

After adding the JavaScript:

1. Click **Save**.
2. Run the application.
3. Open the page containing the Rich Text Editor.
4. Click the **Upload Image** button.
5. Select an image from your local machine.
6. The image will be inserted into the editor.

---

# 🖼️ Expected Result

After implementing the above configuration, the Rich Text Editor will provide an image-upload option.

Users can select an image and insert it directly into the editor content without manually creating an image URL.

---

# ⚙️ Important Technical Note

This implementation uses **Base64 encoding**.

That means the image itself is stored inside the HTML content as a Data URL.

For example:

```html
<p>Employee Dashboard</p>

<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUg..." />
```

### Advantages

* No separate image-upload API is required.
* No external image server is required.
* Simple to implement.
* Useful for small images and lightweight applications.
* Image can be displayed directly as part of the HTML content.

### Considerations

Base64 encoding increases the size of the HTML content.

Therefore, this approach may not be ideal when:

* Users upload very large images.
* A large number of images are stored.
* Rich Text Editor content is saved frequently.
* Database storage size is a concern.
* Images need to be reused independently.
* Image management and security requirements are more complex.

For larger enterprise applications, a better approach can be to upload the image to a database table, object storage, or another file repository and store only the image URL/reference inside the Rich Text Editor.

---

# 🔐 Production Considerations

Before using this approach in a production application, consider implementing:

* Maximum image file size validation
* Allowed image MIME types
* Image dimension validation
* Security validation
* Database/storage limitations
* Image compression
* Proper handling of large Base64 content

For example, you may want to allow only:

```text
image/png
image/jpeg
image/gif
image/webp
```

and restrict the maximum file size according to your application's requirements.

---

# 🎯 Use Cases

This technique can be useful for applications such as:

* 📚 Knowledge Management Systems
* 📖 Documentation Applications
* 📢 Announcement Pages
* 📝 Internal Tools
* 👨‍💼 Employee Portals
* 🐛 Issue/Incident Management
* 📋 Business Process Applications
* 💡 Knowledge Base Applications

---

# ✅ Conclusion

By using a custom CKEditor upload adapter in the Oracle APEX Rich Text Editor, we can enable users to **upload and insert images directly into the editor**.

The solution is lightweight because it uses the browser's `FileReader` API to convert the selected image into a Base64 Data URL.

For simple use cases and smaller images, this can be a convenient approach. For enterprise applications with large or numerous images, consider implementing server-side or object-storage-based image management instead.

I hope this article helps Oracle APEX developers enhance the Rich Text Editor experience in their applications.

If you found this useful, feel free to **star ⭐ the repository, share it, or leave your feedback/comments**.

Happy APEX Development! 🚀
