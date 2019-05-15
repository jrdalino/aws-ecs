# AWS ECS Fargate

## Step 1 Create the ECS Fargate Cluster and CloudWatch Logs Group

### Step 1.1: Create ECS Cluster
What is an ECS Cluster?
- A regional grouping of one or more container instances on which you can run task requests.

```
$ aws ecs create-cluster \
--cluster-name calculator-awsecs
```

### Step 1.2: Create Cloudwatch Logs Group
What is CloudWatch Logs?
- AWS CloudWatch Logs is a service for log collection and analysis Why?
- This is especially important when using AWS Fargate since you will not have access to the server infrastructure where your containers are running.

```
$ aws logs create-log-group \
--log-group-name calculator-awsecs-logs
```

### Step 1.3: Create a Network Load Balancer
Why?
- So we don't directly expose our service to the internet
- Scale-in and scale-out Replace:
- REPLACE_ME_PUBLIC_SUBNET_ONE = subnet-07c8c59e404042290
- REPLACE_ME_PUBLIC_SUBNET_TWO = subnet-031895c836de1c0cb

```
$ aws elbv2 create-load-balancer \
--name calculator-awsecs-nlb \
--scheme internet-facing \
--type network \
--subnets subnet-07c8c59e404042290 subnet-031895c836de1c0cb
```

### Step 1.4: Create a Load Balancer Target Group
Replace:
- vpc-id = REPLACE_ME_VPC_ID = vpc-088a1ae059f28535d

```
$ aws elbv2 create-target-group \
--name calculator-awsecs-targetgroup \
--port 8080 \
--protocol TCP \
--target-type ip \
--vpc-id vpc-088a1ae059f28535d \
--health-check-interval-seconds 10 \
--health-check-path / \
--health-check-protocol HTTP \
--healthy-threshold-count 3 \
--unhealthy-threshold-count 3
```

### Step 1.5: Create a Load Balancer Listener
Replace:
- TargetGroupArn
- load-balancer-arn

```
$ aws elbv2 create-listener \
--default-actions TargetGroupArn=arn:aws:elasticloadbalancing:ap-southeast-1:486051038643:targetgroup/calculator-awsecs-targetgroup/ac7b2d26215b386d,Type=forward \
--load-balancer-arn arn:aws:elasticloadbalancing:ap-southeast-1:486051038643:loadbalancer/net/calculator-awsecs-nlb/2749eba949a7bb50 \
--port 80 \
--protocol TCP
```

### Step 1.6: Creating a Service Linked Role for ECS
```
$ aws iam create-service-linked-role \
--aws-service-name ecs.amazonaws.com
```

### (Optional) Clean up
