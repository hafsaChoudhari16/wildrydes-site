# 🦄 Wild Rydes – Serverless Ride-Sharing Web Application

A full-stack **serverless web application** built using AWS that allows users to create an account, authenticate securely, and request a unicorn ride by selecting a pickup location.

This project demonstrates how multiple AWS services can be integrated to build a scalable, secure, and serverless application without managing traditional backend servers.

---

## 📌 Project Overview

**Wild Rydes** is a unicorn ride-sharing application where authenticated users can request a ride by selecting their pickup location.

The application uses **Amazon Cognito for authentication, API Gateway for the REST API, AWS Lambda for backend processing, and Amazon DynamoDB for data storage.** The frontend is hosted using AWS Amplify, while GitHub is used for source-code management and deployment.

---

## ✨ Features

* 🔐 User registration and authentication
* 📧 Email-based user attributes through Amazon Cognito
* 🗺️ Map-based ride requests
* 🦄 Random unicorn assignment
* 🌐 REST API using Amazon API Gateway
* ⚡ Serverless backend using AWS Lambda
* 💾 Ride data stored in Amazon DynamoDB
* 🔑 Cognito-based API authorization using JWT tokens
* 🛡️ IAM-based permissions and access control
* 🚀 Frontend hosting and deployment using AWS Amplify
* 📦 Source-code management using GitHub

---

## 🏗️ Architecture

The application follows a serverless architecture:

```text
User
  │
  ▼
Web Application
  │
  ▼
AWS Amplify
  │
  ▼
Amazon Cognito
  │
  │ JWT Token
  ▼
Amazon API Gateway
  │
  │ POST /ride
  ▼
AWS Lambda
  │
  ▼
Amazon DynamoDB
```

### Supporting Services

```text
GitHub ───────────────► AWS Amplify
                           │
                           │ Deployment
                           ▼
                        Frontend

AWS IAM ───────────────► AWS Resources
                           │
                           ├── Lambda
                           ├── DynamoDB
                           └── API Gateway
```

---

## ☁️ AWS Services Used

| Service                | Purpose                                                        |
| ---------------------- | -------------------------------------------------------------- |
| **AWS Amplify**        | Hosts and deploys the frontend application                     |
| **Amazon Cognito**     | Handles user authentication and user management                |
| **Amazon API Gateway** | Provides the REST API endpoint and authorization               |
| **AWS Lambda**         | Processes ride requests and contains backend logic             |
| **Amazon DynamoDB**    | Stores ride information                                        |
| **AWS IAM**            | Manages permissions and access between AWS resources           |
| **GitHub**             | Stores source code and integrates with the deployment workflow |

---

## 🔐 Authentication

Amazon Cognito is used to manage users and secure access to the application.

The configured user pool uses:

* **Username** as the sign-in identifier
* **Email** as a required user attribute
* Cognito User Pool authentication
* JWT tokens for authenticated requests

The authenticated user's Cognito username is retrieved by the Lambda function from the API Gateway authorizer.

```javascript
const username =
    event.requestContext.authorizer.claims['cognito:username'];
```

This username is then stored along with the ride information in DynamoDB.

---

## 🌐 API Gateway

Amazon API Gateway exposes the ride-request API.

### Endpoint

```text
POST /ride
```

The API is protected using a **Cognito authorizer**.

The client sends an authentication token with the request:

```text
Authorization: <JWT Token>
```

API Gateway validates the authenticated request before invoking the Lambda function.

---

## ⚡ AWS Lambda

The backend is implemented using:

```text
Runtime: Node.js 24.x
AWS SDK: JavaScript SDK v3
```

The Lambda function is responsible for:

1. Validating the API authorization configuration
2. Generating a unique Ride ID
3. Retrieving the authenticated username
4. Reading the pickup location
5. Selecting a unicorn from the available fleet
6. Saving the ride information to DynamoDB
7. Returning the ride details to the client

### Unicorn Fleet

The application currently uses a sample fleet:

```javascript
const fleet = [
    { Name: 'Angel', Color: 'White', Gender: 'Female' },
    { Name: 'Gil', Color: 'White', Gender: 'Male' },
    { Name: 'Rocinante', Color: 'Yellow', Gender: 'Female' },
];
```

A unicorn is randomly selected for each ride request.

---

## 💾 Amazon DynamoDB

Ride information is stored in a DynamoDB table named:

```text
Rides
```

Each ride record contains:

```text
RideId
User
Unicorn
RequestTime
```

Example:

```json
{
  "RideId": "generated-ride-id",
  "User": "username",
  "Unicorn": {
    "Name": "Angel",
    "Color": "White",
    "Gender": "Female"
  },
  "RequestTime": "2026-08-15T08:00:00.000Z"
}
```

---

## 🔄 Application Workflow

The complete request flow is:

### 1. User Authentication

The user creates an account and signs in through **Amazon Cognito**.

### 2. Ride Request

The authenticated user selects a pickup location on the map.

### 3. API Request

The frontend sends a:

```text
POST /ride
```

request to API Gateway along with the authentication token.

### 4. Authorization

API Gateway uses the Cognito authorizer to validate the user's JWT token.

### 5. Lambda Processing

The request is forwarded to AWS Lambda.

Lambda:

* Retrieves the authenticated user
* Reads the pickup coordinates
* Selects a unicorn
* Generates a Ride ID

### 6. Database Storage

Lambda writes the ride information to the DynamoDB `Rides` table.

### 7. Response

The Lambda function returns the ride information to the frontend.

Example:

```json
{
  "RideId": "generated-ride-id",
  "Unicorn": {
    "Name": "Angel",
    "Color": "White",
    "Gender": "Female"
  },
  "Eta": "30 seconds",
  "Rider": "username"
}
```

---

## 🔑 IAM & Security

AWS IAM is used to control permissions between AWS resources.

The Lambda execution role provides the permissions required for the function to interact with DynamoDB.

The application also uses:

* Amazon Cognito authentication
* JWT-based API authorization
* API Gateway authorization
* IAM-based AWS resource permissions

This follows the principle of giving AWS resources only the permissions required for their operations.

---

## 🔗 GitHub & Amplify

The application source code is maintained in **GitHub**.

AWS Amplify is connected to the application repository to provide frontend hosting and deployment.

```text
GitHub
   │
   │ Code Push
   ▼
AWS Amplify
   │
   │ Deployment
   ▼
Hosted Web Application
```

---

## 🧪 Testing

The Lambda function and API were tested using an authenticated ride request.

### Test Request

```json
{
  "path": "/ride",
  "httpMethod": "POST",
  "headers": {
    "Accept": "*/*",
    "Authorization": "<JWT_TOKEN>",
    "content-type": "application/json"
  },
  "requestContext": {
    "authorizer": {
      "claims": {
        "cognito:username": "the_username"
      }
    }
  },
  "body": "{\"PickupLocation\":{\"Latitude\":47.6174755835663,\"Longitude\":-122.28837066650185}}"
}
```

### Successful Response

```json
{
  "RideId": "generated-ride-id",
  "Unicorn": {
    "Name": "Angel",
    "Color": "White",
    "Gender": "Female"
  },
  "Eta": "30 seconds",
  "Rider": "the_username"
}
```

The generated ride was also verified in the DynamoDB `Rides` table.

---

## 🛠️ Technologies

**Cloud Platform:** AWS
**Frontend Hosting:** AWS Amplify
**Authentication:** Amazon Cognito
**API:** Amazon API Gateway
**Backend:** AWS Lambda
**Runtime:** Node.js 24.x
**Database:** Amazon DynamoDB
**Access Management:** AWS IAM
**SDK:** AWS SDK for JavaScript v3
**Version Control:** Git & GitHub

---

## 📚 Key Learning Outcomes

Through this project, I gained practical experience with:

* Building serverless applications on AWS
* Designing a multi-service cloud architecture
* Implementing authentication using Amazon Cognito
* Securing APIs using JWT authorization
* Building REST APIs with API Gateway
* Developing serverless backend functions with Lambda
* Using DynamoDB as a NoSQL database
* Managing AWS IAM roles and permissions
* Hosting and deploying applications with AWS Amplify
* Connecting GitHub with AWS deployment workflows
* Integrating multiple AWS services into a complete application

---

## 💰 Cost Considerations

The application was developed using AWS Free Tier/available AWS credits where applicable.

AWS resources should be terminated or deleted when the project is no longer being used to prevent unexpected charges.

---

## 🎯 Project Highlights

This project demonstrates a complete serverless workflow:

```text
Authentication
      ↓
API Authorization
      ↓
REST API
      ↓
Serverless Backend
      ↓
NoSQL Database
```

It provides hands-on experience with **AWS cloud services, serverless architecture, authentication, API development, database management, IAM, and deployment automation**.

---

## 👨‍💻 Project

### Wild Rydes – Serverless Ride-Sharing Web Application

A hands-on AWS project demonstrating the integration of **AWS Amplify, Amazon Cognito, API Gateway, Lambda, DynamoDB, IAM, and GitHub** to build a complete serverless web application.

