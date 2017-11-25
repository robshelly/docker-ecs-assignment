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

for TD in stock-price-td weather-td portal-td 
do 
  aws ecs register-task-definition --cli-input-json file://task-definitions/$TD.json
done 

for TD in stock-price weather portal 
do 
aws ecs create-service --cluster $CLUSTER_NAME --service-name $TD-service --task-definition $TD:$REVISION --desired-count 1
done


```
