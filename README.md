# AWS Config Tag Compliance Automation

This project implements an automated AWS Config rule that ensures EC2 instances and RDS databases have the required tags. It includes automatic remediation and email notifications using AWS SES.

## Features

- Automated tag compliance checking for EC2 and RDS resources
- Automatic remediation of missing tags
- Email notifications using AWS SES
- Skip mechanism for resources that shouldn't be tagged
- Detailed logging and error handling
- SSM Parameter Store integration for configuration

## Prerequisites

- AWS Account with appropriate permissions
- Python 3.8 or later
- AWS CLI configured
- Verified SES email address
- Required AWS services:
  - AWS Config
  - AWS Lambda
  - Amazon SES
  - Systems Manager Parameter Store
  - IAM

## Installation

1. Clone the repository:
```bash
git clone <repository-url>
cd <repository-name>
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

3. Configure AWS credentials:
```bash
aws configure
```

## Configuration

### SSM Parameters

Create the following parameters in Systems Manager Parameter Store:

1. Required Tags:
```bash
aws ssm put-parameter \
    --name "/default/required_tags" \
    --value "Environment,Project,Owner" \
    --type String
```

2. Skip Tag Key:
```bash
aws ssm put-parameter \
    --name "/default/skip_tag_key" \
    --value "nochanges" \
    --type String
```

3. Notification Email:
```bash
aws ssm put-parameter \
    --name "/default/notification_email" \
    --value "your-email@domain.com" \
    --type String
```

4. Tag Values (for each required tag):
```bash
aws ssm put-parameter \
    --name "/default/tags/Environment" \
    --value "Production" \
    --type String

aws ssm put-parameter \
    --name "/default/tags/Project" \
    --value "MyProject" \
    --type String

aws ssm put-parameter \
    --name "/default/tags/Owner" \
    --value "TeamName" \
    --type String
```

### IAM Role

Create an IAM role for the Lambda function with the following permissions:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeTags",
                "ec2:CreateTags",
                "rds:ListTagsForResource",
                "rds:AddTagsToResource",
                "ssm:GetParameter",
                "config:PutEvaluations",
                "ses:SendEmail",
                "ses:SendRawEmail",
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "*"
        }
    ]
}
```

## Deployment

1. Create a ZIP file of the Lambda function:
```bash
zip -r lambda_function.zip config.py
```

2. Create the Lambda function:
```bash
aws lambda create-function \
    --function-name tag-compliance-checker \
    --runtime python3.8 \
    --role arn:aws:iam::<account-id>:role/tag-compliance-role \
    --handler config.lambda_handler \
    --zip-file fileb://lambda_function.zip
```

3. Configure AWS Config rule:
```bash
aws config put-config-rule \
    --config-rule '{
        "ConfigRuleName": "required-tags",
        "Description": "Checks if required tags are present on EC2 and RDS resources",
        "Source": {
            "Owner": "CUSTOM_LAMBDA",
            "SourceIdentifier": "arn:aws:lambda:<region>:<account-id>:function:tag-compliance-checker"
        },
        "Scope": {
            "ComplianceResourceTypes": [
                "AWS::EC2::Instance",
                "AWS::RDS::DBInstance"
            ]
        }
    }'
```

## Usage

The function will automatically:
1. Check for required tags on EC2 and RDS resources
2. Add missing tags if found
3. Send email notifications for:
   - Tag updates
   - Compliance status
   - Errors
   - Skipped resources

### Email Notifications

You will receive emails for:
- Tag updates: When missing tags are added
- Compliance status: When resources are compliant
- Errors: When processing fails
- Skipped resources: When resources have the skip tag

## Testing

1. Test the Lambda function:
```bash
aws lambda invoke \
    --function-name tag-compliance-checker \
    --payload '{"invokingEvent": "{\"configurationItem\":{\"resourceType\":\"AWS::EC2::Instance\",\"resourceId\":\"i-1234567890abcdef0\"}}", "resultToken": "test-token"}' \
    output.json
```

2. Check CloudWatch Logs for execution details:
```bash
aws logs get-log-events \
    --log-group-name /aws/lambda/tag-compliance-checker \
    --log-stream-name <log-stream-name>
```

## Troubleshooting

### Common Issues

1. **SES Email Not Sending**
   - Verify sender email in SES
   - Check IAM permissions
   - Ensure recipient email is valid

2. **Missing Tags Not Added**
   - Check SSM parameters exist
   - Verify IAM permissions
   - Check resource type is supported

3. **Lambda Execution Errors**
   - Check CloudWatch Logs
   - Verify IAM role permissions
   - Check SSM parameter values

### Logging

Logs are available in CloudWatch Logs under:
- Log Group: `/aws/lambda/tag-compliance-checker`
- Log Stream: `<execution-id>`

## Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Support

For support, please:
1. Check the troubleshooting guide
2. Review CloudWatch Logs
3. Create an issue in the repository

## Authors

- Guru Santosh Vyakaranam

## Acknowledgments

- AWS Config
- AWS Lambda
- Amazon SES
- Systems Manager Parameter Store
