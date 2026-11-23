# EventRadar

## Run locally

1. Copy `.env.example` to `.env` and set your Groq and ElevenLabs credentials.
2. Create the DynamoDB favorites table described below and configure `AWS_REGION` and `DYNAMODB_TABLE_NAME`.
3. Configure AWS credentials using the AWS SDK default credential chain (for example, an AWS profile or an IAM role). Do not put AWS access keys in browser code.
4. Run `npm start` and open `http://localhost:3000`.

The ElevenLabs API key and voice ID are read only by the server. Event details, not the user's profile, are sent to ElevenLabs to create a voice digest.

## Public deployment safeguards

The chat and prep-brief routes share a limit of 20 requests per IP address per 15 minutes. Voice generation is limited to 5 requests per IP address per 15 minutes. The server trusts one reverse-proxy hop to identify client IPs behind Render. These in-memory limits are intended for a single-instance demo; they reset when the server restarts and do not replace provider spending limits or authentication.

## DynamoDB favorites

Create a DynamoDB table with:

- Partition key: `browserId` (String)
- Sort key: `eventId` (String)
- Billing mode: on-demand (`PAY_PER_REQUEST`)

For example, using the AWS CLI:

```powershell
aws dynamodb create-table `
  --table-name eventradar-favorites `
  --attribute-definitions AttributeName=browserId,AttributeType=S AttributeName=eventId,AttributeType=S `
  --key-schema AttributeName=browserId,KeyType=HASH AttributeName=eventId,KeyType=RANGE `
  --billing-mode PAY_PER_REQUEST `
  --region ap-south-1
```

The server's AWS identity needs `dynamodb:Query`, `dynamodb:UpdateItem`, and `dynamodb:DeleteItem` access to this table. Favorite records contain only the anonymous browser ID, event ID, and creation timestamp; profile details are not stored in DynamoDB.

The browser ID is a random identifier stored in local browser storage, not an authentication mechanism. This is suitable only for non-sensitive favorites; use an authenticated identity before storing private user data.
