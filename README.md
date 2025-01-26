# AWS Step Functions EC2 Instance Management

AWS Step Functions state machine definitions for starting and stopping EC2 instances based on specified tags.

## State Machine Definition Files

- `start_ec2_instances.json`: Start stopped EC2 instances with specified tags
- `stop_ec2_instances.json`: Stop running EC2 instances with specified tags

## Deployment

### 1. Create IAM Role for State Machine (First time only)

First, create an IAM role for Step Functions.
Replace `YOUR_ROLE_NAME` with your preferred role name (e.g., `StepFunctionsEC2ManagementRole`):

```bash
aws iam create-role \
  --role-name YOUR_ROLE_NAME \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {
          "Service": "states.amazonaws.com"
        },
        "Action": "sts:AssumeRole"
      }
    ]
  }'
```

Next, attach EC2 operation permissions to the created role.
Replace `YOUR_POLICY_NAME` with your preferred policy name (e.g., `EC2ManagementPolicy`):

```bash
aws iam put-role-policy \
  --role-name YOUR_ROLE_NAME \
  --policy-name YOUR_POLICY_NAME \
  --policy-document '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "ec2:DescribeInstances",
          "ec2:StartInstances",
          "ec2:StopInstances"
        ],
        "Resource": "*"
      }
    ]
  }'
```

### 2. Create State Machines

Create a state machine for starting EC2 instances. Replace `YOUR_ROLE_NAME` with the role name created above:

```bash
aws stepfunctions create-state-machine \
  --name EC2StartInstances \
  --definition file://start_ec2_instances.json \
  --role-arn arn:aws:iam::YOUR_ACCOUNT_ID:role/YOUR_ROLE_NAME \
  --type STANDARD
```

Create a state machine for stopping EC2 instances:

```bash
aws stepfunctions create-state-machine \
  --name EC2StopInstances \
  --definition file://stop_ec2_instances.json \
  --role-arn arn:aws:iam::YOUR_ACCOUNT_ID:role/YOUR_ROLE_NAME \
  --type STANDARD
```

Note: 
- Replace `YOUR_ACCOUNT_ID` with your actual AWS account ID
- Replace `YOUR_ROLE_NAME` with the IAM role name created above

## Usage

When executing the state machine, specify input JSON like this:

```json
{
  "TagName": "Environment",  // Tag name to search for
  "TagValue": "Development" // Tag value to search for
}
```

### Command Line Examples

To start EC2 instances:

```bash
aws stepfunctions start-execution \
  --state-machine-arn arn:aws:states:REGION:YOUR_ACCOUNT_ID:stateMachine:EC2StartInstances \
  --input '{"TagName": "Environment", "TagValue": "Development"}'
```

To stop EC2 instances:

```bash
aws stepfunctions start-execution \
  --state-machine-arn arn:aws:states:REGION:YOUR_ACCOUNT_ID:stateMachine:EC2StopInstances \
  --input '{"TagName": "Environment", "TagValue": "Development"}'
```

Note: Replace `REGION` and `YOUR_ACCOUNT_ID` with your actual values.

## Cleanup

### 1. Delete State Machines

Delete the EC2 start instances state machine:

```bash
aws stepfunctions delete-state-machine \
  --state-machine-arn arn:aws:states:REGION:YOUR_ACCOUNT_ID:stateMachine:EC2StartInstances
```

Delete the EC2 stop instances state machine:

```bash
aws stepfunctions delete-state-machine \
  --state-machine-arn arn:aws:states:REGION:YOUR_ACCOUNT_ID:stateMachine:EC2StopInstances
```

### 2. Delete IAM Role (Optional)

First, delete the role policy:

```bash
aws iam delete-role-policy \
  --role-name YOUR_ROLE_NAME \
  --policy-name YOUR_POLICY_NAME
```

Then, delete the IAM role:

```bash
aws iam delete-role \
  --role-name YOUR_ROLE_NAME
```

Note:
- Replace `REGION` with your AWS region
- Replace `YOUR_ACCOUNT_ID` with your actual AWS account ID
- Replace `YOUR_ROLE_NAME` with the IAM role name you created
- Replace `YOUR_POLICY_NAME` with the policy name you created

## Limitations

- This tool only supports starting and stopping EC2 instances based on a single tag key-value pair
- State machines are executed sequentially, which means if you have many instances to process, it may take some time to complete
- AWS Step Functions Standard workflow type is used, which may incur costs based on the number of state transitions

## References

- [AWS Step Functions Developer Guide](https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html)
- [AWS Step Functions Service Integrations](https://docs.aws.amazon.com/step-functions/latest/dg/supported-services-awssdk.html)
- [AWS EC2 Instance States](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html)
