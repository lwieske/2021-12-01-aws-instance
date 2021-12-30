---
marp: true
---

<!-- _class: invert -->

## Amazon Web Services - AWS

* AWS is a subsidiary of Amazon, the American multinational technology company. 
  
* AWS provides on-demand cloud computing platforms and APIs to customers.

* AWS offers over 200 services from redundant data centers distributed globally.

* Amazon Elastic Compute Cloud (Amazon EC2) - offers over 400 instance types.

* Popular services like Dropbox, Netflix, Foursquare oder Reddit use AWS.

---

## Elastic Compute Cloud - EC2

AWS spans 81 availability zones within 25 geographic regions.

```
        _,--',   _._.--._____
 .--.--';_'-.', ";_      _.,-'
.'--'.  _.'    {`'-;_ .-.>.'
      '-:_      )  / `' '=.
        ) >     {_/,     /~)
        |/               `^ .'
```

---

## VPC - the networking core of AWS

* A virtual private cloud (VPC) is a virtual network dedicated to your AWS account.

* A VPC is logically isolated from (your) other virtual networks in AWS.

* You can launch your AWS resources, such as Amazon EC2 instances, into your VPC.

* When you create a VPC, you must specify a range of IPv4 addresses for the VPC.

---

## VPC (II)

```
VPC_ID=`aws ec2 create-vpc \
    --cidr-block 10.0.0.0/2 \
    --query Vpc.VpcId \
    --output text`

aws ec2 modify-vpc-attribute \
    --enable-dns-hostnames \
    --vpc-id ${VPC_ID} 
aws ec2 modify-vpc-attribute \
    --enable-dns-support \
    --vpc-id ${VPC_ID}
```

---

## Subnet - network topology within the VPC

* You can add one or more subnets in each Availability Zone within a Region.

* You can launch AWS resources, such as EC2 instances, into a specific subnet.

* A subnet is a range of IP addresses, given as CIDR blocks, in your VPC.

* The CIDR block for the subnet is a subset of the VPC CIDR block.

* Each subnet resides entirely within one single Availability Zone.

---

## Subnet (II)

By launching instances in separate Availability Zones, you can protect your
applications from the failure of a single zone.

```
PUBLIC_SUBNET_ID=`aws ec2 create-subnet \
    --cidr-block 10.0.1.0/24 \
    --vpc-id ${VPC_ID} \
    --query "Subnet.SubnetId" \
    --output text`
```

---

## Internet Gateway - Accessing EC2 instances in the Internet

* An internet gateway allows communication between your VPC and the internet.

* An internet gateway serves two purposes:

  * provides a target in your VPC route tables for internet-routable traffic.
  
  * performs network address translation (NAT) for instances with public ips.

* An internet gateway supports IPv4 and IPv6 traffic.

* There's no additional charge for an internet gateway.

---

## Internet Gateway (II)

```
IGW_ID=`aws ec2 create-internet-gateway \
    --query InternetGateway.InternetGatewayId \
    --output text`
```

* Public subnets are associated with a route to an internet gateway.

* Private subnets do not have any routes to an internet gateway.

---

## Enabling Internet Access (I)

* Create an internet gateway and attach it to your VPC.

```
aws ec2 attach-internet-gateway \
    --vpc-id ${VPC_ID} \
    --internet-gateway-id ${IGW_ID}
```

---

## Enabling Internet Access (II)

* Add a route to your subnet's route table directs internet-bound traffic.

```
RTB_ID=`aws ec2 create-route-table \
    --vpc-id ${VPC_ID} \
    --query RouteTable.RouteTableId \
    --output text`

aws ec2 create-route \
    --route-table-id ${RTB_ID} \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id ${IGW_ID}

aws ec2 associate-route-table \
    --subnet-id ${PUBLIC_SUBNET_ID} \
    --route-table-id ${RTB_ID}
```

---

## Enabling Internet Access (III)

* Ensure that instances have a globally unique IP address.

```
aws ec2 modify-subnet-attribute \
    --subnet-id ${PUBLIC_SUBNET_ID} \
    --map-public-ip-on-launch
```

---

## Enabling Internet Access (IV)

* Ensure that security groups allow relevant traffic.

```
aws ec2 create-security-group \
    --group-name ${SECURITY_GROUP_NAME} \
    --description "SSH access for AUTHORIZED_EXTERNAL_IP" \
    --vpc-id ${VPC_ID}

AUTHORIZED_EXTERNAL_IP=`curl https://checkip.amazonaws.com`

aws ec2 authorize-security-group-ingress \
    --group-name ${SECURITY_GROUP_NAME} \
    --protocol tcp \
    --port 22 \
    --cidr ${AUTHORIZED_EXTERNAL_IP}
```

---

<!-- _class: invert -->

## Launching the EC2 instance

* AMIs(Amazon Machine Images) offer operating systems for instances.
  
* Instances are launched into a subnet in a virtual private cloud (VPC)

  * ... with a security group allowing network traffic

  * ... and instrumented with a key pair for ssh access.

* Instance come up pending initially and change to running state shortly after.

```
aws ec2 run-instances \
    --image-id resolve:ssm:/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 \
    --count 1 \
    --instance-type t2.micro \
    --key-name ${KEY_NAME} \
    --security-group-ids ${SECURITY_GROUP_NAME} \
    --subnet-id ${PUBLIC_SUBNET_ID}
```

---

<!-- _class: invert -->

## Before Getting Started

* Using the AWS CLI requires authentication in a secure way.

* Using AWS IAM may secure your AWS resources and limits financial risks.

* **RTFM**: Please, read the documentation on AWS IAM + AWS EC2 carefully.

* Here, a temporary user's credentials were set as environment variables.

* **Caution**: wait for instance and ssh to come up ...

---

## General Purpose Instances

* Amazon EC2 provides a wide selection of **instance types** for different use cases.

* Instance types comprise varying combinations of (CPU, RAM, storage, network ...)
  
* Each instance type includes one or more **instance sizes**, allowing you to scale.

* General purpose instances provide a balanced mix of resources ...

* Amazon’s T instance family is optimized for “burstable” workloads.

* low CPU utilization and some periods of high CPU activity (bursts).

---

## T Instance Family

| Region    | Instance  | pCPU              | vCPU | vRAM | Price USD/hour |
|:---------:|:---------:|:-----------------:|:----:|:----:|---------------:|
| Frankfurt | t2.micro  | Intel Xeon        | 1    | 1    | 0.0134         |
| Frankfurt | t3.micro  | Intel Skylake     | 2    | 1    | 0.012          |
| Frankfurt | t3a.micro | AMD EPYC 7000     | 2    | 1    | 0.0108         |
| Frankfurt | t4g.micro | ARM/AWS Graviton2 | 2    | 1    | 0.0096         |

* Source: https://www.instance-pricing.com (March 2021)
* No guarantees/liabilities. Conduct your own research.

---

<!-- _class: invert -->

## Amazon Linux 2

* Amazon Linux 2 is a server operating system from Amazon Web Services (AWS).

* AL2 long-term support includes security updates and bug fixes for 5 years.

* AL2 is provided without charge as an Amazon Machine Image (AMI) for EC2.

* AL2 is available as container/virtual images downloads for free as well.

* These images can be used for on-premises development and testing.



---

## AL2 - Software

* Extras in AL2 provide bleeding edge software on a stable base of AL2.

* AL2 packages and configurations provide tight integration with AWS services.

* AL2 comes among others with many AWS tools (e.g. AWS CLI) and cloud-init.

* AL2 includes the widely adopted systemd init system - (was upstart before).

* AL2 includes a LTS Kernel tuned for enhanced performance on Amazon EC2.

---

## AL2 - Security

* AL2 limits remote access by using SSH key pairs and by disabling remote root login.

* AL2 reduces the number of non-critical packages which are installed on an instance.

* So, it limits exposure to potential security vulnerabilities and attack vectors.

* Security updates rated critical/important are automatically applied on initial boot.

* AL2 allows for kernel live patching (kernel patches without reboot or downtime).

---

## Cleanup

* Remove all AWS resources created without breaking their dependencies:

```
aws ec2 terminate-instances      --instance-ids           ${INSTANCE_ID}
aws ec2 delete-key-pair          --key-name               ${KEY_NAME}
aws ec2 wait instance-terminated --instance-ids           ${INSTANCE_ID}
aws ec2 delete-security-group    --group-id               ${SECURITY_GROUP_ID}
aws ec2 delete-route             --destination-cidr-block 0.0.0.0/0            --route-table-id ${RTB_ID}
aws ec2 detach-internet-gateway  --internet-gateway-id    ${IGW_ID}            --vpc-id ${VPC_ID}
aws ec2 delete-internet-gateway  --internet-gateway-id    ${IGW_ID}
aws ec2 delete-subnet            --subnet-id              ${PUBLIC_SUBNET_ID}
aws ec2 delete-route-table       --route-table-id         ${RTB_ID}
aws ec2 delete-vpc               --vpc-id                 ${VPC_ID}
```