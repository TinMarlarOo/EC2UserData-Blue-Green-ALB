# aws-EC2UserData & Blue-Green Deployment using Application Loadbalancer

# Steps: 

## 1. Create common VPC teamplate and deploy in sg
- [vpc.yaml](./Templates/vpc.yaml)

```bash 
aws cloudformation create-stack --stack-name sgpvpc --template-body file://vpc.yaml --parameters ParameterKey='VPCCIDR',ParameterValue='192.168.0.0/16' ParameterKey='PublicSubnet1CIDR',ParameterValue='192.168.1.0/24' ParameterKey='PublicSubnet2CIDR',ParameterValue='192.168.2.0/24' ParameterKey='PublicSubnet3CIDR',ParameterValue='192.168.3.0/24' ParameterKey='RegionCode',ParameterValue='sgp' ParameterKey='AZ1Code',ParameterValue='sgpaz1' ParameterKey='AZ2Code',ParameterValue='sgpaz2' ParameterKey='AZ3Code',ParameterValue='sgpaz3'
```
## 2. Create security group for ALB and instance
-[vpc-securitygroup.yaml](./Templates/vpc-securitygroup.yaml)

```bash
aws cloudformation create-stack --stack-name sgpvpc-securitygroup --template-body file://sgpvpc-securitygroup.yaml --parameters ParameterKey='vpcStackName',ParameterValue='sgpvpc'
```

## 3. Create instance 1 web server
-[public-instance-v1.yaml](./Templates/public-instance-v1.yaml)

```bash
aws cloudformation create-stack --stack-name public-instance-v1 --template-body file://public-instance-v1.yaml --parameters ParameterKey='vpcStackName',ParameterValue='sgpvpc' ParameterKey='vpcSecurityGroupStackName',ParameterValue='sgpvpc-securitygroup' ParameterKey='appVersion',ParameterValue='v1'
```
## 4. Create instance 1 web server
-[public-instance-v2.yaml](./Templates/public-instance-v2.yaml)

```bash
aws cloudformation create-stack --stack-name public-instance-v2 --template-body file://public-instance-v2.yaml --parameters ParameterKey='vpcStackName',ParameterValue='sgpvpc' ParameterKey='vpcSecurityGroupStackName',ParameterValue='sgpvpc-securitygroup' ParameterKey='appVersion',ParameterValue='v2'
```

## 5.Setup Application loadbalancer
-[applicationloadbalancer.yaml](./Templates/applicationloadbalancer.yaml)

```bash
aws cloudformation create-stack --stack-name applicationloadbalancer --template-body file://applicationloadbalancer.yaml
``` 

<br>

### Test result
```bash
➜  CloudFormation while sleep 0.9; do curl -k "applicationloadbalancer-102036986.ap-southeast-1.elb.amazonaws.com"; done
<html><h1 align='center'><p style='color:blue'>m y g e n t o c l o u d. c o m - app v1</p></h1></html>
<html><h1 align='center'><p style='color:green'>m y g e n t o c l o u d. c o m - app v2</p></h1></html>
<html><h1 align='center'><p style='color:blue'>m y g e n t o c l o u d. c o m - app v1</p></h1></html>
<html><h1 align='center'><p style='color:green'>m y g e n t o c l o u d. c o m - app v2</p></h1></html>
```
## Delete stack
```bash
#!/bin/sh
for i in applicationloadbalancer public-instance-v2 public-instance-v1 sgpvpc-securitygroup vpc; 
 do aws cloudformation delete-stack --stack-name $i; 
 sleep 50;
 done
 ```
  ~ vi delete-stack.sh
## need to run the following command
  ./delete-stack.sh:wqi





