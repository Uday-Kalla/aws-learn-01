Here’s a **sample static website creation script** (basic HTML, CSS, and optional JS) that you can use with your CloudFront + S3 deployment:

---

## 📁 Folder: `site/`

```
site/
├── index.html
├── style.css
└── script.js (optional)
```

---

### ✅ 1. `site/index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Welcome to My CloudFront Website</title>
  <link rel="stylesheet" href="style.css" />
</head>
<body>
  <div class="container">
    <h1>Hello from AWS CloudFront 🚀</h1>
    <p>This static site is deployed using S3 + CloudFront + Terraform + GitHub Actions.</p>
    <button onclick="showMessage()">Click Me</button>
    <p id="message"></p>
  </div>
  <script src="script.js"></script>
</body>
</html>
```

---

### 🎨 2. `site/style.css`

```css
body {
  font-family: Arial, sans-serif;
  background: #f7f9fc;
  color: #333;
  text-align: center;
  padding: 50px;
}

.container {
  max-width: 600px;
  margin: auto;
  background: white;
  padding: 30px;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

button {
  padding: 10px 20px;
  background: #0077cc;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
```

---

### 🧠 3. `site/script.js` (optional)

```javascript
function showMessage() {
  document.getElementById("message").innerText = "Thanks for clicking! 😊";
}
```

---

## ✅ How to Use

* Put all 3 files inside the `site/` folder in your project.
* Terraform and GitHub Actions will automatically sync these to S3 and serve them via CloudFront.

---

Would you like me to zip this up or add this to the existing Terraform PDF guide?
