# AWS CloudFormation Deep Dive

A comprehensive guide to advanced AWS CloudFormation concepts, stack management, and enterprise patterns.

## What is AWS CloudFormation?
- AWS native Infrastructure as Code service
- Uses JSON or YAML templates to define infrastructure  
- Automatic dependency resolution and rollback capabilities
- Deep integration with all AWS services
- Stack-based resource management

## Template Structure

### Basic Template Anatomy
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'My CloudFormation template'

Parameters:
  EnvironmentName:
    Type: String
    Default: Development
    Description: Environment name prefix

Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-0c55b159cbfafe1d0
    us-west-2:
      AMI: ami-0d70546e43a941d70

Conditions:
  CreateProdResources: !Equals [!Ref EnvironmentName, Production]

Resources:
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName}-VPC

Outputs:
  VPCId:
    Description: VPC ID
    Value: !Ref MyVPC
    Export:
      Name: !Sub ${AWS::StackName}-VPC-ID
```

## Advanced Features

### Nested Stacks
Parent stack template:
```yaml
Resources:
  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/network-template.yaml
      Parameters:
        EnvironmentName: !Ref EnvironmentName
        
  ApplicationStack:
    Type: AWS::CloudFormation::Stack
    DependsOn: NetworkStack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/app-template.yaml
      Parameters:
        VPCId: !GetAtt NetworkStack.Outputs.VPCId
```

### Custom Resources with Lambda
```yaml
Resources:
  CustomResourceFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: !Sub ${AWS::StackName}-custom-resource
      Runtime: python3.9
      Handler: index.lambda_handler
      Code:
        ZipFile: |
          import boto3
          import cfnresponse
          
          def lambda_handler(event, context):
              try:
                  if event['RequestType'] == 'Create':
                      # Custom creation logic
                      response_data = {'Message': 'Resource created successfully'}
                      cfnresponse.send(event, context, cfnresponse.SUCCESS, response_data)
                  elif event['RequestType'] == 'Delete':
                      # Custom deletion logic
                      cfnresponse.send(event, context, cfnresponse.SUCCESS, {})
              except Exception as e:
                  cfnresponse.send(event, context, cfnresponse.FAILED, {})

  MyCustomResource:
    Type: Custom::MyResource
    Properties:
      ServiceToken: !GetAtt CustomResourceFunction.Arn
      CustomProperty: !Ref SomeParameter
```

### Cross-Stack References
```yaml
# In Network Stack
Outputs:
  VPCId:
    Value: !Ref VPC
    Export:
      Name: !Sub ${AWS::StackName}-VPC-ID
      
  PrivateSubnetIds:
    Value: !Join [',', [!Ref PrivateSubnet1, !Ref PrivateSubnet2]]
    Export:
      Name: !Sub ${AWS::StackName}-Private-Subnets

# In Application Stack
Resources:
  ApplicationLoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Scheme: internal
      Subnets: !Split [',', !ImportValue NetworkStack-Private-Subnets]
```

## Advanced Patterns

### Multi-Environment Template
```yaml
Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, staging, prod]
    Default: dev

Mappings:
  EnvironmentMap:
    dev:
      InstanceType: t2.micro
      MinSize: 1
      MaxSize: 2
    staging:
      InstanceType: t2.small
      MinSize: 1
      MaxSize: 3
    prod:
      InstanceType: t3.medium
      MinSize: 2
      MaxSize: 10

Resources:
  AutoScalingGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      MinSize: !FindInMap [EnvironmentMap, !Ref Environment, MinSize]
      MaxSize: !FindInMap [EnvironmentMap, !Ref Environment, MaxSize]
      LaunchTemplate:
        LaunchTemplateId: !Ref LaunchTemplate
        Version: !GetAtt LaunchTemplate.LatestVersionNumber

  LaunchTemplate:
    Type: AWS::EC2::LaunchTemplate
    Properties:
      LaunchTemplateData:
        InstanceType: !FindInMap [EnvironmentMap, !Ref Environment, InstanceType]
        ImageId: !Ref LatestAmiId
```

### Blue-Green Deployment Pattern
```yaml
Parameters:
  BlueWeight:
    Type: Number
    Default: 100
    MinValue: 0
    MaxValue: 100
  GreenWeight:
    Type: Number
    Default: 0
    MinValue: 0
    MaxValue: 100

Conditions:
  BlueActive: !Not [!Equals [!Ref BlueWeight, 0]]
  GreenActive: !Not [!Equals [!Ref GreenWeight, 0]]

Resources:
  BlueTargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Condition: BlueActive
    Properties:
      Name: blue-targets
      # ... other properties

  GreenTargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Condition: GreenActive
    Properties:
      Name: green-targets
      # ... other properties

  LoadBalancerListener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      DefaultActions:
        - Type: forward
          ForwardConfig:
            TargetGroups:
              - !If
                - BlueActive
                - TargetGroupArn: !Ref BlueTargetGroup
                  Weight: !Ref BlueWeight
                - !Ref AWS::NoValue
              - !If
                - GreenActive  
                - TargetGroupArn: !Ref GreenTargetGroup
                  Weight: !Ref GreenWeight
                - !Ref AWS::NoValue
```

## Stack Management

### Stack Policies
Prevent accidental updates to critical resources:
```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "Update:*",
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Principal": "*", 
      "Action": "Update:Delete",
      "Resource": "LogicalResourceId/ProductionDatabase"
    }
  ]
}
```

### Change Sets
```bash
# Create change set to preview changes
aws cloudformation create-change-set \
  --stack-name my-stack \
  --change-set-name my-change-set \
  --template-body file://template.yaml \
  --parameters ParameterKey=Environment,ParameterValue=prod

# Review change set
aws cloudformation describe-change-set \
  --stack-name my-stack \
  --change-set-name my-change-set

# Execute change set
aws cloudformation execute-change-set \
  --stack-name my-stack \
  --change-set-name my-change-set
```

### Stack Drift Detection
```bash
# Start drift detection
aws cloudformation detect-stack-drift --stack-name my-stack

# Check drift status
aws cloudformation describe-stack-drift-detection-status \
  --stack-drift-detection-id <detection-id>

# Get drift details
aws cloudformation describe-stack-resource-drifts \
  --stack-name my-stack
```

## Best Practices

### Template Organization
```
cloudformation/
├── templates/
│   ├── networking/
│   │   ├── vpc.yaml
│   │   └── security-groups.yaml
│   ├── compute/
│   │   ├── ec2.yaml
│   │   └── autoscaling.yaml
│   └── storage/
│       ├── s3.yaml
│       └── rds.yaml
├── parameters/
│   ├── dev.json
│   ├── staging.json
│   └── prod.json
└── scripts/
    ├── deploy.sh
    └── validate.sh
```

### Parameter Validation
```yaml
Parameters:
  DatabasePassword:
    Type: String
    NoEcho: true
    MinLength: 8
    MaxLength: 41
    AllowedPattern: '[a-zA-Z0-9]*'
    ConstraintDescription: Must contain only alphanumeric characters
    
  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small
      - t2.medium
    ConstraintDescription: Must be a valid EC2 instance type
```

### Tagging Strategy
```yaml
Resources:
  MyResource:
    Type: AWS::EC2::Instance
    Properties:
      Tags:
        - Key: Name
          Value: !Sub ${AWS::StackName}-instance
        - Key: Environment
          Value: !Ref Environment
        - Key: Project
          Value: !Ref ProjectName
        - Key: Owner
          Value: !Ref Owner
        - Key: CostCenter  
          Value: !Ref CostCenter
```

## Troubleshooting

### Common Issues and Solutions

#### Stack Rollback
```bash
# Continue rollback from failed state
aws cloudformation continue-update-rollback --stack-name my-stack

# Cancel update (if stack is in UPDATE_IN_PROGRESS)
aws cloudformation cancel-update-stack --stack-name my-stack
```

#### Resource Creation Failures
```yaml
# Add DeletionPolicy to prevent accidental deletion
Resources:
  Database:
    Type: AWS::RDS::DBInstance
    DeletionPolicy: Snapshot
    Properties:
      # ... properties
      
  S3Bucket:
    Type: AWS::S3::Bucket
    DeletionPolicy: Retain
```

#### Debugging with CloudFormation Events
```bash
# Monitor stack events in real-time
aws cloudformation describe-stack-events \
  --stack-name my-stack \
  --query 'StackEvents[?Timestamp>=`2024-01-01T00:00:00Z`]' \
  --output table
```

## Integration with CI/CD

### AWS CLI Deployment Script
```bash
#!/bin/bash
STACK_NAME=$1
TEMPLATE_FILE=$2
PARAMETERS_FILE=$3
ENVIRONMENT=$4

# Validate template
aws cloudformation validate-template --template-body file://$TEMPLATE_FILE

# Deploy stack
aws cloudformation deploy \
  --stack-name $STACK_NAME \
  --template-file $TEMPLATE_FILE \
  --parameter-overrides file://$PARAMETERS_FILE \
  --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM \
  --tags Environment=$ENVIRONMENT
```

## Exercises & Next Steps

- Exercise: Create a nested stack architecture for a 3-tier application
- Next: Implement custom resources for complex configurations  
- Advanced: Build a complete CI/CD pipeline using CloudFormation

## Resources
- [AWS CloudFormation Documentation](https://docs.aws.amazon.com/cloudformation/)
- [CloudFormation Template Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-reference.html)
- [AWS CloudFormation Best Practices](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/best-practices.html)