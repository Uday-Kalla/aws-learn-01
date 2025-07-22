✅ Sure! Here's a **Patient Service** and **Appointment Service** example — **two separate Node.js Lambda functions**.

---

## ✅ **1. patient-service/index.js**

```javascript
exports.handler = async (event) => {
    // Example data - list of 5 patients
    const patients = [
        { id: 1, name: "Alice", age: 30 },
        { id: 2, name: "Bob", age: 25 },
        { id: 3, name: "Charlie", age: 35 },
        { id: 4, name: "Diana", age: 28 },
        { id: 5, name: "Evan", age: 40 }
    ];

    return {
        statusCode: 200,
        body: JSON.stringify({ patients })
    };
};
```

---

## ✅ **2. appointment-service/index.js**

```javascript
exports.handler = async (event) => {
    // Example data - list of 5 appointments
    const appointments = [
        { id: 101, patientId: 1, doctor: "Dr. Smith", date: "2025-07-23" },
        { id: 102, patientId: 2, doctor: "Dr. John", date: "2025-07-24" },
        { id: 103, patientId: 3, doctor: "Dr. Sara", date: "2025-07-25" },
        { id: 104, patientId: 4, doctor: "Dr. Watson", date: "2025-07-26" },
        { id: 105, patientId: 5, doctor: "Dr. Kate", date: "2025-07-27" }
    ];

    return {
        statusCode: 200,
        body: JSON.stringify({ appointments })
    };
};
```

---

## ✅ **Next Steps**

You can:

1. **Zip each file separately for Lambda:**

```bash
cd patient-service
zip patient-service.zip index.js

cd ../appointment-service
zip appointment-service.zip index.js
```

2. **Deploy each zip as a separate Lambda function.**
3. **Expose via API Gateway** if you want API endpoints:

* `/patients` -> triggers `patient-service`
* `/appointments` -> triggers `appointment-service`

---

## ✅ **Bonus**

If you'd like, I can:

* Provide **Terraform code to deploy both Lambda functions**
* Provide **API Gateway routes for both**
* Provide **step-by-step deployment instructions**

👉 **Would you like the full Terraform + API Gateway setup for these two services?**
