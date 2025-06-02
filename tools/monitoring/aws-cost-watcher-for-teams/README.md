# AWS Cost Watcher for Microsoft Teams

[![🇯🇵 日本語](https://img.shields.io/badge/%F0%9F%87%AF%F0%9F%87%B5-日本語-white)](./README-ja.md) [![🇺🇸 English](https://img.shields.io/badge/%F0%9F%87%BA%F0%9F%87%B8-English-white)](./README.md)

An AWS cost monitoring system that automatically posts weekly AWS cost summaries to Microsoft Teams channels. This tool is part of the aws-devops-toolkit repository and is located in `tools/monitoring/aws-cost-watcher`. This solution is inspired by [this article](https://dev.classmethod.jp/articles/aws-cost-watcher-with-sfn-jsonata/) and has been adapted from Slack to Microsoft Teams with customized messaging.

## Features

- 📊 **Weekly Cost Monitoring**: Automatically tracks AWS costs over the past 7 days
- 🚨 **Threshold Alerts**: Sends angry notifications when costs exceed your defined threshold
- 📈 **Top 5 Services**: Shows the top 5 AWS services by cost
- 🤖 **Automated Scheduling**: Runs daily at 10:00 AM JST
- 👥 **Teams Integration**: Posts formatted messages directly to your Microsoft Teams channel

## Architecture

This solution uses the following AWS services:
- **AWS Step Functions**: Orchestrates the cost monitoring workflow
- **AWS Cost Explorer API**: Retrieves cost and usage data
- **AWS Chatbot**: Integrates with Microsoft Teams
- **Amazon SNS**: Publishes notifications
- **Amazon EventBridge Scheduler**: Triggers daily executions

## Prerequisites

- AWS CLI configured with appropriate permissions
- Microsoft Teams channel where you want to receive notifications
- Teams App ID, Tenant ID, and Channel ID

## Getting Teams Information

To deploy this solution, you'll need the following Microsoft Teams information:

### 1. Team ID
1. Open Microsoft Teams
2. Navigate to your team
3. Click on the three dots (...) next to the team name
4. Select "Get link to team"
5. Extract the Team ID from the URL (GUID format)

### 2. Teams Tenant ID
1. Go to Azure Portal
2. Navigate to Azure Active Directory
3. Copy the Tenant ID from the overview page

### 3. Teams Channel ID
1. Open Microsoft Teams
2. Navigate to the specific channel
3. Click on the three dots (...) next to the channel name
4. Select "Get link to channel"
5. Extract the Channel ID from the URL (URL-encoded format)

## Deployment

1. Navigate to the tool directory:
```bash
cd tools/monitoring/aws-cost-watcher
```

2. Deploy the CloudFormation stack:
```bash
aws cloudformation deploy \
  --template-file aws-cost-watcher-for-teams.yaml \
  --stack-name aws-cost-watcher-teams \
  --parameter-overrides \
    Project=your-project \
    Env=prod \
    TeamId=your-team-id \
    TeamsChannelID=your-channel-id \
    TeamsTenantId=your-tenant-id \
    AngryThreshold=150 \
  --capabilities CAPABILITY_NAMED_IAM
```

## Parameters

| Parameter | Description | Default | Required |
|-----------|-------------|---------|----------|
| `Project` | Project name for resource naming | `sample` | No |
| `Env` | Environment name | `dev` | No |
| `NameSuffix` | Suffix for resource names | `aws-cost-watcher` | No |
| `TeamId` | Microsoft Teams Team ID (GUID format) | - | Yes |
| `TeamsChannelID` | Teams Channel ID (URL-encoded) | - | Yes |
| `TeamsTenantId` | Microsoft Teams Tenant ID (GUID format) | - | Yes |
| `AngryThreshold` | Weekly cost threshold in USD for angry notifications | `150` | No |

## Message Format

The system sends two types of messages based on your cost threshold:

### Normal Message (✅😄)
When costs are below the threshold, you'll receive a calm notification with:
- Current week's total cost
- Top 5 services by cost
- Collection period
- Cost threshold information

### Angry Message (❌🤬)
When costs exceed the threshold, you'll receive an urgent notification with the same information but with alert emojis and urgent messaging.

## Customization

You can customize the following aspects:

### Schedule
Modify the `ScheduleExpression` in the CloudFormation template:
```yaml
ScheduleExpression: "cron(00 10 * * ? *)"  # Daily at 10:00 AM JST
```

### Cost Threshold
Adjust the `AngryThreshold` parameter when deploying or update the stack.

### Message Format
The message format is defined in the Step Functions definition using JSONata expressions. You can modify the `title` and `description` fields to customize the appearance.

## Monitoring and Troubleshooting

### Check Step Functions Execution
1. Go to AWS Step Functions console
2. Find your state machine: `{Project}-{Env}-aws-cost-watcher-sfn`
3. Review execution history and logs

### Verify Chatbot Configuration
1. Go to AWS Chatbot console
2. Check your Microsoft Teams configuration
3. Ensure the SNS topic is properly configured

### Cost Explorer Permissions
Ensure the Step Functions execution role has the `ce:GetCostAndUsage` permission.

## Cost Considerations

This solution incurs minimal costs:
- Step Functions: ~$0.025 per 1,000 executions
- SNS: ~$0.50 per 1 million notifications
- Cost Explorer API: $0.01 per request

For daily execution, monthly costs should be under $1.

## Security

The solution follows AWS security best practices:
- IAM roles with minimal required permissions
- No hardcoded credentials
- Encrypted SNS topics (optional, can be enabled)

## Contributing

This tool is part of the aws-devops-toolkit repository. To contribute:

1. Fork the aws-devops-toolkit repository
2. Create a feature branch
3. Make your changes in the `tools/monitoring/aws-cost-watcher` directory
4. Submit a pull request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- Inspired by the original [AWS Cost Watcher article](https://dev.classmethod.jp/articles/aws-cost-watcher-with-sfn-jsonata/)
- Thanks to the AWS community for Step Functions and JSONata examples