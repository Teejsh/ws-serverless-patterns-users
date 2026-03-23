# Users Service

A serverless REST API for managing user data, built with AWS SAM, API Gateway, Lambda, DynamoDB, and Cognito.

## Architecture

- **API Gateway**: REST API with Lambda token authorizer and access logging
- **UsersFunction**: Lambda function handling all CRUD operations (Python 3.11)
- **AuthorizerFunction**: Lambda authorizer validating Cognito JWT tokens
- **DynamoDB**: `Users` table with `userid` as the partition key
- **Cognito**: User Pool with hosted UI, `apiAdmins` group for admin privileges
- **CloudWatch**: Alarms for API 5XX errors, Lambda errors/throttles, and an application dashboard
- **SNS**: Alarms topic for notifications

## Authorization

All API routes are protected by a Lambda token authorizer backed by Cognito:

- Regular users can only access their own `/users/{userid}` resources (GET, PUT, DELETE)
- Users in the `apiAdmins` Cognito group have full access to all `/users` routes

## Prerequisites

- [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)
- Python 3.11
- AWS credentials configured

## Installation

```bash
pip install -r requirements.txt
sam build
```

## API Endpoints

| Method | Endpoint | Description | Admin Only |
|--------|----------|-------------|------------|
| GET | `/users` | Get all users | Yes |
| POST | `/users` | Create new user | Yes |
| GET | `/users/{userid}` | Get user by ID | No |
| PUT | `/users/{userid}` | Update user by ID | No |
| DELETE | `/users/{userid}` | Delete user by ID | No |

## Deployment

```bash
# First deployment
sam deploy --guided

# Subsequent deployments
sam deploy
```

## Local Testing

```bash
sam local invoke UsersFunction -e events/event-get-all-users.json
sam local invoke UsersFunction -e events/event-get-user-by-id.json
sam local invoke UsersFunction -e events/event-post-user.json
sam local invoke UsersFunction -e events/event-put-user.json
sam local invoke UsersFunction -e events/event-delete-user-by-id.json
```

## Authentication

Get a Cognito ID token to use as the `Authorization` header:

```bash
aws cognito-idp initiate-auth \
  --auth-flow USER_PASSWORD_AUTH \
  --client-id <UserPoolClient> \
  --auth-parameters USERNAME=<email>,PASSWORD=<password> \
  --query 'AuthenticationResult.IdToken' \
  --output text
```

## Request/Response Examples

**Create User (POST /users):**
```json
{
  "name": "<name>",
  "email": "<email>"
}
```

**Response:**
```json
{
  "userid": "<generated-uuid>",
  "name": "<name>",
  "email": "<email>",
  "timestamp": "2024-01-15T10:30:00.000000"
}
```

## Environment Variables

| Variable | Description |
|----------|-------------|
| `USERS_TABLE` | DynamoDB table name (auto-configured) |
| `USER_POOL_ID` | Cognito User Pool ID (auto-configured) |
| `APPLICATION_CLIENT_ID` | Cognito App Client ID (auto-configured) |
| `ADMIN_GROUP_NAME` | Cognito admin group name (default: `apiAdmins`) |

## Stack Outputs

| Output | Description |
|--------|-------------|
| `APIEndpoint` | API Gateway endpoint URL |
| `UserPool` | Cognito User Pool ID |
| `UserPoolClient` | Cognito App Client ID |
| `CognitoLoginURL` | Hosted UI login URL |
| `AlarmsTopic` | SNS topic ARN for alarms |
| `DashboardURL` | CloudWatch dashboard URL |

## Testing

```bash
pytest tests/unit
pytest tests/integration
```

## Clean Up

```bash
sam delete
```

## License

MIT-0
