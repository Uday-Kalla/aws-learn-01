### ✅ **Terraform Setup for AWS Lambda + API Gateway (Step by Step)**

> This will create:

1. **Lambda Function**
2. **API Gateway** to trigger Lambda
3. **IAM Role for Lambda**

---

## ✅ **Step 1: Create Project Structure**

```bash
mkdir lambda-terraform-demo
cd lambda-terraform-demo
touch main.tf variables.tf outputs.tf lambda_function.zip
```

---

## ✅ **Step 2: Sample Lambda Function Code (index.js)**

```javascript
exports.handler = async (event) => {
    const name = event.queryStringParameters.name || "Guest";
    return {
        statusCode: 200,
        body: JSON.stringify(`Hello, ${name}! Welcome to Lambda via API Gateway.`)
    };
};
```

---

## ✅ **Step 3: Zip the Lambda Code**

```bash
zip lambda_function.zip index.js
```

---

## ✅ **Step 4: Terraform Configuration**

### **main.tf**

```hcl
provider "aws" {
  region = "ap-south-1"  # Change to your region
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

resource "aws_iam_role_policy_attachment" "lambda_policy" {
  role       = aws_iam_role.lambda_exec_role.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}

resource "aws_lambda_function" "hello_lambda" {
  filename         = "lambda_function.zip"
  function_name    = "HelloLambda"
  role             = aws_iam_role.lambda_exec_role.arn
  handler          = "index.handler"
  runtime          = "nodejs18.x"
  source_code_hash = filebase64sha256("lambda_function.zip")
}

resource "aws_apigatewayv2_api" "http_api" {
  name          = "HelloLambdaAPI"
  protocol_type = "HTTP"
}

resource "aws_apigatewayv2_integration" "lambda_integration" {
  api_id           = aws_apigatewayv2_api.http_api.id
  integration_type = "AWS_PROXY"
  integration_uri  = aws_lambda_function.hello_lambda.invoke_arn
}

resource "aws_apigatewayv2_route" "default_route" {
  api_id    = aws_apigatewayv2_api.http_api.id
  route_key = "GET /hello"
  target    = "integrations/${aws_apigatewayv2_integration.lambda_integration.id}"
}

resource "aws_lambda_permission" "api_gateway_permission" {
  statement_id  = "AllowAPIGatewayInvoke"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.hello_lambda.function_name
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

## ✅ **Step 5: Initialize and Apply Terraform**

```bash
terraform init
terraform plan
terraform apply
```

---

## ✅ **Step 6: Test the API Endpoint**

Once applied, Terraform will output an API URL like:

```
https://xyz123.execute-api.ap-south-1.amazonaws.com
```

Test via:

```bash
curl https://xyz123.execute-api.ap-south-1.amazonaws.com/hello?name=John
```

You should get:

```json
"Hello, John! Welcome to Lambda via API Gateway."
```

---

## ✅ **Summary**

| Step | Description              |
| ---- | ------------------------ |
| 1    | Write Lambda Code        |
| 2    | Zip the code             |
| 3    | Create Terraform files   |
| 4    | `terraform init` & apply |
| 5    | Test via API URL         |

---

Let me know if you'd like:

* A **PDF guide of these steps**
* A **version with S3-backed Lambda code deployment**
