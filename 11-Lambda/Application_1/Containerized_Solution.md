✅ Sure! Here's how to **containerize both Patient Service and Appointment Service as AWS Lambda Container Images**, deploy via **Terraform**, and expose via **API Gateway**.

---

## ✅ **Step 1: Project Structure**

```
lambda-container-demo/
│
├── patient-service/
│   ├── Dockerfile
│   └── index.js
│
├── appointment-service/
│   ├── Dockerfile
│   └── index.js
│
├── main.tf
├── variables.tf
├── outputs.tf
```

---

## ✅ **Step 2: Node.js Code**

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

## ✅ **Step 3: Dockerfile for Each Service**

### **Common Dockerfile**

```Dockerfile
FROM public.ecr.aws/lambda/nodejs18.x

COPY index.js ./

CMD ["index.handler"]
```

Place this in both `patient-service/` and `appointment-service/`.

---

## ✅ **Step 4: Build and Push Docker Images**

### **Create ECR Repositories**

```bash
aws ecr create-repository --repository-name patient-service
aws ecr create-repository --repository-name appointment-service
```

### **Build and Push Images**

#### Patient Service:

```bash
cd patient-service
docker build -t patient-service .
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.ap-south-1.amazonaws.com
docker tag patient-service:latest <aws_account_id>.dkr.ecr.ap-south-1.amazonaws.com/patient-service:latest
docker push <aws_account_id>.dkr.ecr.ap-south-1.amazonaws.com/patient-service:latest
```

#### Appointment Service:

```bash
cd ../appointment-service
docker build -t appointment-service .
docker tag appointment-service:latest <aws_account_id>.dkr.ecr.ap-south-1.amazonaws.com/appointment-service:latest
docker push <aws_account_id>.dkr.ecr.ap-south-1.amazonaws.com/appointment-service:latest
```

---

## ✅ **Step 5: Terraform Configuration Example**

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
  package_type  = "Image"
  image_uri     = "<aws_account_id>.dkr.ecr.ap-south-1.amazonaws.com/patient-service:latest"
  function_name = "PatientService"
  role          = aws_iam_role.lambda_exec_role.arn
}

resource "aws_lambda_function" "appointment_service" {
  package_type  = "Image"
  image_uri     = "<aws_account_id>.dkr.ecr.ap-south-1.amazonaws.com/appointment-service:latest"
  function_name = "AppointmentService"
  role          = aws_iam_role.lambda_exec_role.arn
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

## ✅ **Step 6: Deploy**

1. **Run Terraform**

```bash
terraform init
terraform apply
```

2. Terraform will output the API Gateway URL like:

```
https://xyz123.execute-api.ap-south-1.amazonaws.com
```

3. **Test APIs**

```bash
curl https://xyz123.execute-api.ap-south-1.amazonaws.com/patients
curl https://xyz123.execute-api.ap-south-1.amazonaws.com/appointments
```

---

## ✅ **Summary**

| Endpoint        | Description                       |
| --------------- | --------------------------------- |
| `/patients`     | Runs Patient Lambda Container     |
| `/appointments` | Runs Appointment Lambda Container |

---

✅ Let me know if you'd like:

* A **ready-made zipped project with all files**
* A **step-by-step PDF guide**
* **GitHub repo-style structure for CI/CD integration**
