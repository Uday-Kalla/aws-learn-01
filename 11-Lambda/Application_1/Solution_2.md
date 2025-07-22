✅ Here's a **complete Terraform setup to deploy both the Patient Service and Appointment Service as AWS Lambda functions** with **API Gateway HTTP endpoints.**

---

## ✅ **Project Structure**

```
lambda-terraform-microservices/
│
├── appointment-service/
│   └── index.js
├── patient-service/
│   └── index.js
├── main.tf
├── variables.tf
├── outputs.tf
```

---

## ✅ **Lambda Service Codes**

### `patient-service/index.js`

```javascript
exports.handler = async (event) => {
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

### `appointment-service/index.js`

```javascript
exports.handler = async (event) => {
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

## ✅ **Zip Lambda Packages**

```bash
cd patient-service
zip ../patient-service.zip index.js
cd ../appointment-service
zip ../appointment-service.zip index.js
cd ..
```

---

## ✅ **Terraform Code**

### `main.tf`

```hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_iam_role" "lambda_exec_role" {
  name = "lambda_exec_role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "lambda.amazonaws.com"
      }
    }]
  })
}

resource "aws_iam_role_policy_attachment" "lambda_policy_attach" {
  role       = aws_iam_role.lambda_exec_role.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}

resource "aws_lambda_function" "patient_service" {
  filename         = "patient-service.zip"
  function_name    = "PatientService"
  role             = aws_iam_role.lambda_exec_role.arn
  handler          = "index.handler"
  runtime          = "nodejs18.x"
  source_code_hash = filebase64sha256("patient-service.zip")
}

resource "aws_lambda_function" "appointment_service" {
  filename         = "appointment-service.zip"
  function_name    = "AppointmentService"
  role             = aws_iam_role.lambda_exec_role.arn
  handler          = "index.handler"
  runtime          = "nodejs18.x"
  source_code_hash = filebase64sha256("appointment-service.zip")
}

resource "aws_apigatewayv2_api" "http_api" {
  name          = "MicroservicesAPI"
  protocol_type = "HTTP"
}

resource "aws_apigatewayv2_integration" "patient_integration" {
  api_id           = aws_apigatewayv2_api.http_api.id
  integration_type = "AWS_PROXY"
  integration_uri  = aws_lambda_function.patient_service.invoke_arn
}

resource "aws_apigatewayv2_integration" "appointment_integration" {
  api_id           = aws_apigatewayv2_api.http_api.id
  integration_type = "AWS_PROXY"
  integration_uri  = aws_lambda_function.appointment_service.invoke_arn
}

resource "aws_apigatewayv2_route" "patient_route" {
  api_id    = aws_apigatewayv2_api.http_api.id
  route_key = "GET /patients"
  target    = "integrations/${aws_apigatewayv2_integration.patient_integration.id}"
}

resource "aws_apigatewayv2_route" "appointment_route" {
  api_id    = aws_apigatewayv2_api.http_api.id
  route_key = "GET /appointments"
  target    = "integrations/${aws_apigatewayv2_integration.appointment_integration.id}"
}

resource "aws_lambda_permission" "allow_api_gateway_patient" {
  statement_id  = "AllowAPIGatewayInvokePatient"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.patient_service.function_name
  principal     = "apigateway.amazonaws.com"
  source_arn    = "${aws_apigatewayv2_api.http_api.execution_arn}/*/*"
}

resource "aws_lambda_permission" "allow_api_gateway_appointment" {
  statement_id  = "AllowAPIGatewayInvokeAppointment"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.appointment_service.function_name
  principal     = "apigateway.amazonaws.com"
  source_arn    = "${aws_apigatewayv2_api.http_api.execution_arn}/*/*"
}

resource "aws_apigatewayv2_stage" "default_stage" {
  api_id      = aws_apigatewayv2_api.http_api.id
  name        = "$default"
  auto_deploy = true
}

output "api_endpoint" {
  value = aws_apigatewayv2_api.http_api.api_endpoint
}
```

---

## ✅ **Deploy with Terraform**

1. Initialize:

```bash
terraform init
```

2. Plan:

```bash
terraform plan
```

3. Apply:

```bash
terraform apply
```

---

## ✅ **Test APIs**

After `terraform apply`, the **API URL will be printed**, e.g.:

```
https://xyz123.execute-api.ap-south-1.amazonaws.com
```

### ✅ Test Patient Service:

```bash
curl https://xyz123.execute-api.ap-south-1.amazonaws.com/patients
```

### ✅ Test Appointment Service:

```bash
curl https://xyz123.execute-api.ap-south-1.amazonaws.com/appointments
```

---

## ✅ **Summary**

| Endpoint        | Description                  |
| --------------- | ---------------------------- |
| `/patients`     | Returns list of patients     |
| `/appointments` | Returns list of appointments |

---

✅ Let me know if you want:

* The **complete zipped folder with all files**
* A **step-by-step PDF guide**
* **CloudWatch logging setup**
