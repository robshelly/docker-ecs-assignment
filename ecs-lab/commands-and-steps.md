```bash
############################################################
#
# List of AWS CLI commands used to createa infrastructure
#
#
############################################################

######### AWS Container Exercise ##########


# Create a cluster, no instance created, just an mpty named cluster
aws ecs create-cluster --cluster-name $CLUSTER_NAME

# Launch ECS Optimized instance
aws ec2 run-instances --image-id $AMI_ID --count 1 --instance-type t2.micro --key-name $KEY_NAME --security-groups $HTTPSSH_SEC_GROUP $VPC_SEC_GROUP --iam-instance-profile Name=$ECS_INSTANCE_ROLE --user-data file://files/user-data.txt --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value='$INSTANCE_NAME'}]'

# Create task definition
aws ecs register-task-definition --cli-input-json file://simple-app-task-def.json

# Run the task
aws ecs run-task --cluster $CLUSTER_NAME --task-definition $TASK_DEF:$REVISION --count 1

###### SUCCESS ######

########## Scaling Container Exercise ##########

# Create LB
aws elbv2 create-load-balancer --type application --name $LB_NAME --security-groups $HTTPSSH_SEC_GROUP_ID $VPC_SEC_GROUP_ID --subnets $SUBNET_A_ID $SUBNET_B_ID $SUBNET_C_ID

# Create Target Group
aws elbv2 create-target-group --name $TG_NAME --protocol HTTP --port 80 --vpc-id $VPC_ID --target-type instance --health-check-protocol HTTP --health-check-path "/index.php"

# Register Targets <Target Group ARN  returned by above command)
aws elbv2 register-targets --target-group-arn $TG_ARN --targets Id=$INSTANCE_ID

# Add listener
aws elbv2 create-listener --load-balancer-arn $LB_ARN --protocol HTTP --port 80 --default-actions Type=forward,TargetGroupArn=$TG_ARN

# Create service
aws ecs create-service --cluster $CLUSTER_NAME --service-name $SERVICE_NAME --task-definition $TASK_DEF:$REVISION --load-balancers targetGroupArn=$TG_ARN,containerName=$CONTAINER_NAME,containerPort=80 --desired-count 2 --role $ECS_SERVICE_ROLE

# Register Scalable Target
aws application-autoscaling register-scalable-target --service-namespace ecs --resource-id service/$CLUSTER_NAME/$SERVICE_NAME --scalable-dimension ecs:service:DesiredCount --min-capacity 1 --max-capacity 3 --role-arn $APP_AUTOSCALING_ROLE_ARN

### Create scaling policies and Alarms
# Scale Out Policy
aws application-autoscaling put-scaling-policy --policy-name $SCALE_OUT_POLICY_NAME  --service-namespace ecs --resource-id service/$CLUSTER_NAME/$SERVICE_NAME --scalable-dimension ecs:service:DesiredCount --policy-type StepScaling --step-scaling-policy-configuration="AdjustmentType=ChangeInCapacity,StepAdjustments=[{ScalingAdjustment=0,MetricIntervalLowerBound=0}],MetricAggregationType=Average,Cooldown=60"
# Scale Out Alarm
aws cloudwatch put-metric-alarm --alarm-name $SCALE_OUT_ALARM_NAME --metric-name CPUUtilization --namespace AWS/ECS --statistic Average --dimensions "Name=ClusterName,Value=php-app Name=ServiceName,Value=php-app-service" --period 60 --evaluation-periods 60 --comparison-operator GreaterThanOrEqualToThreshold --threshold 75 --unit Percent --actions-enabled --alarm-actions $SCALE_OUT_POLICY_ARN

# Scale In Policy
aws application-autoscaling put-scaling-policy --policy-name $SCALE_IN_POLICY_NAME  --service-namespace ecs --resource-id service/$CLUSTER_NAME/$SERVICE_NAME --scalable-dimension ecs:service:DesiredCount --policy-type StepScaling --step-scaling-policy-configuration="AdjustmentType=ChangeInCapacity,StepAdjustments=[{ScalingAdjustment=0,MetricIntervalLowerBound=0}],MetricAggregationType=Average,Cooldown=60"
# Scale In Alarm
aws cloudwatch put-metric-alarm --alarm-name $SCALE_IN_ALARM_NAME --metric-name CPUUtilization --namespace AWS/ECS --statistic Average --dimensions "Name=ClusterName,Value=php-app Name=ServiceName,Value=php-app-service" --period 60 --evaluation-periods 60 --comparison-operator LessThanOrEqualToThreshold --threshold 25 --unit Percent --actions-enabled --alarm-actions $SCALE_IN_POLICY_ARN

```