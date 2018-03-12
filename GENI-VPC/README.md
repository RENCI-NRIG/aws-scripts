- Bastion host has EIP, other 2 hosts use NAT Gateway to get out to the Internet
- Use ssh key agent (ssh -A ec2-user@public-ip) to transitively login to hosts
- We place all subnets into one AZ, because bastion instance has interfaces in each private subnet (just because) and this requires all three subnets to be in same AZ
- Remember that each route table has default routes within VPC. Can't be modified.


- Sending commands to multiple instances:
	- by id aws ssm send-command --instance-ids i-0b387c665628f5f9b i-02e2a213adfa03bab i-0e6edec45f97ede23 --document-name "AWS-RunShellScript" --comment "IP config" --parameters commands=ifconfig --output text
	- by tag  aws ssm send-command --targets "Key=tag:aws:cloudformation:stack-name,Values=GENIStack" --document-name "AWS-RunShellScript" --comment "IP config" --parameters commands=ifconfig --output text
		- Note that AWS help page incorrectly says use ';' to separate Key and values"

- getting output
	- aws ssm list-command-invocations --command-id 70d6fe00-e586-4e24-90ef-df83645c1886 --details
	- or status aws ssm get-command-invocation --command-id 70d6fe00-e586-4e24-90ef-df83645c1886 --instance-id mi-0f3982491aca3f88e

- Seem unable to reuse the role created by cloudformation to issue tokens for GENI instances - had to create another role based on managed policy

- Create role http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/systems-manager-managedinstances.html
	- aws iam create-role --role-name SSMServiceRole --assume-role-policy-document file://SSMService-Trust.json
	- aws iam attach-role-policy --role-name SSMServiceRole --policy-arn arn:aws:iam::aws:policy/service-role/AmazonEC2RoleforSSM

- Create activation to get code
	- aws ssm create-activation --default-instance-name MyXoServers --iam-role SSMServiceRole --registration-limit 10 

- Install the ssm agent in the slice and pass it the activation code

- Creating inventory associations:
	- aws ssm create-association --name AWS-GatherSoftwareInventory --targets  "Key=instanceids,Values=mi-0ce69bc6ddf91460b,mi-0f3982491aca3f88e,mi-07b59d248d9ff8b6f" --schedule-expression "cron(0 0/30 * 1/1 * ? *)" --parameters networkConfig=Enabled,windowsUpdates=Disabled,applications=Enabled
- Listing inventory
	- list associations aws ssm list-associations
	- list inventory aws ssm list-inventory-entries --instance-id mi-0f3982491aca3f88e --type-name "AWS:Application"
		- also can use type AWS:Network


