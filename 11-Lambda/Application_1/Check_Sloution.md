✅ **You can check the output of the deployed Lambda APIs (patients and appointments) in multiple ways.** Here's how:

---

## ✅ **Step 1: After Terraform Apply**

After running:

```bash
terraform apply
```

You'll see an **API Gateway endpoint output**, like:

```
api_endpoint = https://xyz123.execute-api.ap-south-1.amazonaws.com
```

---

## ✅ **Step 2: Test Using cURL**

### ➡️ **To test Patient Service**

```bash
curl https://xyz123.execute-api.ap-south-1.amazonaws.com/patients
```

You should see:

```json
{
  "patients": [
    { "id": 1, "name": "Alice", "age": 30 },
    { "id": 2, "name": "Bob", "age": 25 },
    ...
  ]
}
```

### ➡️ **To test Appointment Service**

```bash
curl https://xyz123.execute-api.ap-south-1.amazonaws.com/appointments
```

Output:

```json
{
  "appointments": [
    { "id": 101, "patientId": 1, "doctor": "Dr. Smith", "date": "2025-07-23" },
    ...
  ]
}
```

---

## ✅ **Step 3: Test via Browser**

Simply open in browser:

* `https://xyz123.execute-api.ap-south-1.amazonaws.com/patients`
* `https://xyz123.execute-api.ap-south-1.amazonaws.com/appointments`

You'll see the JSON output directly.

---

## ✅ **Step 4: Monitor Logs in AWS CloudWatch**

1. Go to **AWS Console > CloudWatch > Logs > Log groups**.
2. Check for:

   * `/aws/lambda/PatientService`
   * `/aws/lambda/AppointmentService`
3. You can see:

   * Input request data
   * Console logs from Lambda
   * Any errors if occurred

> ✅ If your Lambda has `console.log()`, it appears here.

---

## ✅ **Bonus: Test via Postman**

1. Open **Postman**.
2. Set method to **GET**.
3. URL: `https://xyz123.execute-api.ap-south-1.amazonaws.com/patients`
4. Click **Send** — see JSON response.

---

## ✅ **Summary**

| Method         | Tool        |
| -------------- | ----------- |
| Command Line   | `curl`      |
| Browser        | Direct URL  |
| Postman        | API Testing |
| AWS CloudWatch | Logs        |

---

Let me know if you want me to generate a **PDF guide** with these steps!
