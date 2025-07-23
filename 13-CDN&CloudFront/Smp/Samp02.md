Got it! Here's a **sample static website for a "Medical Appointment Booking"** — tailored for use with AWS S3 + CloudFront + Terraform deployment.

---

## 🏥 Folder: `site/`

```
site/
├── index.html
├── style.css
└── script.js
```

---

### ✅ 1. `site/index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>MedicoCare | Book Your Appointment</title>
  <link rel="stylesheet" href="style.css" />
</head>
<body>
  <div class="container">
    <h1>🩺 MedicoCare</h1>
    <p>Book your appointment with top doctors in minutes</p>

    <form id="appointmentForm">
      <input type="text" placeholder="Full Name" required />
      <input type="email" placeholder="Email Address" required />
      <input type="date" required />
      <select required>
        <option value="">Select Department</option>
        <option value="general">General Physician</option>
        <option value="cardiology">Cardiology</option>
        <option value="dermatology">Dermatology</option>
      </select>
      <button type="submit">Book Appointment</button>
    </form>

    <p id="confirmation"></p>
  </div>

  <script src="script.js"></script>
</body>
</html>
```

---

### 🎨 2. `site/style.css`

```css
body {
  font-family: 'Segoe UI', sans-serif;
  background: #e8f0fe;
  margin: 0;
  padding: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.container {
  background: white;
  padding: 30px 40px;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  text-align: center;
  width: 100%;
  max-width: 400px;
}

h1 {
  margin-bottom: 10px;
  color: #0a66c2;
}

p {
  margin-bottom: 20px;
}

form input,
form select,
form button {
  width: 100%;
  padding: 10px;
  margin: 8px 0;
  border-radius: 5px;
  border: 1px solid #ccc;
  font-size: 16px;
}

form button {
  background-color: #0a66c2;
  color: white;
  cursor: pointer;
  border: none;
  transition: 0.3s ease;
}

form button:hover {
  background-color: #004a99;
}
```

---

### 🧠 3. `site/script.js`

```javascript
document.getElementById("appointmentForm").addEventListener("submit", function (e) {
  e.preventDefault();
  document.getElementById("confirmation").innerText = "✅ Your appointment has been booked!";
});
```

---

## 🔄 How to Use

* Place this inside your `site/` folder
* GitHub Actions will sync it to S3
* Terraform handles S3 + CloudFront + policy

---

Would you like me to:

1. **Zip** the site files and share?
2. **Include these files** in the PDF with Terraform and CI/CD?
3. Add **custom domain and SSL setup** via Route 53 & ACM?

Let me know how you'd like to proceed.
