\# AWS Resource Scheduling Manual Override

This Lambda function provides manual override capabilities for scheduling AWS resources (EC2, EKS, RDS, and LightSail) using EventBridge rules. It includes audit logging and timezone support.

\## Features

\- Manual override scheduling for multiple AWS resources:

\- EC2 Instances

\- EKS Clusters

\- RDS Databases

\- LightSail Instances

\- Timezone-aware scheduling

\- EventBridge rule creation

\- Audit logging

\- DynamoDB integration

\- Support for multiple environments and projects

\## Prerequisites

\- AWS Account with appropriate permissions

\- Python 3.8 or later

\- AWS CLI configured

\- Required AWS services:

\- AWS Lambda

\- Amazon EventBridge

\- Amazon DynamoDB

\- IAM

\## Required AWS Resources

\### DynamoDB Tables

1\. \*\*CalendarEvents Table\*\*

\- Primary Key: \`SN\` (Number)

\- Sort Key: \`Date\` (String)

\- Required attributes:

\- \`Resource-ID\`

\- \`Env\`

\- \`Project\`

\- \`Account-ID\`

\- \`Is-Stop\`

\- \`Event\`

\- \`Time-Zone\`

2\. \*\*Audit Logs Table\*\*

\- Required attributes:

\- \`Account\_ID\`

\- \`Region\`

\- \`Project\`

\- \`Env\`

\- \`Resource\_Type\`

\- \`Resource\_ID\`

\- \`Timezone\`

\- \`From\_Date\`

\- \`To\_Date\`

\- \`Time\`

\- \`EventbridgeRuleName\`

\- \`Action\`

\- \`Status\`

\- \`User\`

\### Lambda Functions

Required Lambda functions:

\- \`start-stop-ec2\`

\- \`mgcsw-eks-scaling\`

\- \`mgcsw-rds-scheduling\`

\- \`Light-Sail-Lambda\`

\## Configuration

\### config.json

Create a \`config.json\` file with the following structure:

\`\`\`json

{

"TIMEZONE\_MAPPING": {

"UTC": "UTC",

"EST": "America/New\_York",

"PST": "America/Los\_Angeles",

"IST": "Asia/Kolkata"

},

"AWS\_REGION\_TIMEZONE\_MAPPING": {

"us-east-1": "America/New\_York",

"us-west-2": "America/Los\_Angeles",

"ap-south-1": "Asia/Kolkata"

}

}

\`\`\`

\### IAM Role

Create an IAM role for the Lambda function with the following permissions:

\`\`\`json

{

"Version": "2012-10-17",

"Statement": \[

{

"Effect": "Allow",

"Action": \[

"dynamodb:Scan",

"dynamodb:UpdateItem",

"dynamodb:PutItem",

"events:PutRule",

"events:PutTargets",

"lambda:AddPermission"

\],

"Resource": "\*"

}

\]

}

\`\`\`

\## Usage

\### Lambda Function Input

\`\`\`json

{

"FromDate": "2024-03-20",

"ToDate": "2024-03-21",

"Env": "prod",

"Project": "myproject",

"Resource-ID": "i-1234567890abcdef0",

"Account-ID": "123456789012",

"Time": "09:00",

"Time-Zone": "UTC",

"Action": "start",

"Is\_Stop": "true",

"UserEmail": "user@example.com",

"resource\_type": "EC2"

}

\`\`\`

\### Supported Resource Types

\- \`EC2\`: EC2 instances

\- \`EKS\`: EKS clusters

\- \`RDS\`: RDS databases

\- \`LightSail\`: LightSail instances

\- \`ALL\`: All supported resource types

\### Timezone Support

The function supports multiple timezones through the \`TIMEZONE\_MAPPING\` configuration. Common timezones include:

\- UTC

\- EST (America/New\_York)

\- PST (America/Los\_Angeles)

\- IST (Asia/Kolkata)

\## Function Flow

1\. \*\*Input Validation\*\*

\- Validates required parameters

\- Checks resource type

\- Verifies timezone

2\. \*\*Calendar Events Update\*\*

\- Updates DynamoDB records

\- Handles different resource types

\- Maintains audit trail

3\. \*\*EventBridge Rule Creation\*\*

\- Creates scheduled rules

\- Sets up Lambda targets

\- Configures timezone-aware scheduling

4\. \*\*Audit Logging\*\*

\- Records all actions

\- Tracks user modifications

\- Maintains history

\## Error Handling

The function includes comprehensive error handling for:

\- Invalid inputs

\- DynamoDB operations

\- EventBridge rule creation

\- Timezone conversions

\- Resource type validation

\## Logging

Logs are available in CloudWatch Logs with the following information:

\- Input parameters

\- Operation status

\- Error details

\- Resource updates

\- EventBridge rule creation

\## Testing

\### Local Testing

1\. Install dependencies:

\`\`\`bash

pip install -r requirements.txt

\`\`\`

2\. Run tests:

\`\`\`bash

python -m pytest tests/

\`\`\`

\### AWS Testing

1\. Deploy the function:

\`\`\`bash

aws lambda update-function-code \\

\--function-name manual-override \\

\--zip-file fileb://function.zip

\`\`\`

2\. Test with sample event:

\`\`\`bash

aws lambda invoke \\

\--function-name manual-override \\

\--payload file://test-event.json \\

response.json

\`\`\`

\## Monitoring

Monitor the function through:

\- CloudWatch Logs

\- DynamoDB tables

\- EventBridge rules

\- Audit logs

\## Troubleshooting

\### Common Issues

1\. \*\*Timezone Conversion Errors\*\*

\- Verify timezone in config.json

\- Check time format

\- Validate timezone mapping

2\. \*\*DynamoDB Update Failures\*\*

\- Check IAM permissions

\- Verify table structure

\- Validate input data

3\. \*\*EventBridge Rule Creation Issues\*\*

\- Check Lambda permissions

\- Verify rule name format

\- Validate cron expression

\## Contributing

1\. Fork the repository

2\. Create a feature branch

3\. Commit your changes

4\. Push to the branch

5\. Create a Pull Request

\## License

This project is licensed under the MIT License - see the LICENSE file for details.

\## Support

For support:

1\. Check the troubleshooting guide

2\. Review CloudWatch Logs

3\. Create an issue in the repository

\## Authors

\- Your Name

\- Your Team

\## Acknowledgments

\- AWS Lambda

\- Amazon EventBridge

\- Amazon DynamoDB

\- AWS IAM