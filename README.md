<!-- PROJECT LOGO -->
<br />
<p align="center">
  <a href="https://github.com/CreditSaisonIndia/oneaboveall/blob/CLS-6162/README.md">
    <img src="images/logo.png" alt="Logo" width="80" height="80">
  </a>

  <h3 align="center">ONE-ABOVE-ALL</h3>

  <p align="center">
    Cloud Infrastructure of Credit Saison written in yaml nested stack templates.
    <br />
    <a href="https://github.com/CreditSaisonIndia/oneaboveall/blob/CLS-6162/README.md"><strong>Explore the docs »</strong></a>
    <br />
    <br />
    <a href="https://github.com/CreditSaisonIndia/oneaboveall/issues">Report Bug</a>
    ·
    <a href="https://github.com/CreditSaisonIndia/oneaboveall/issues">Request Feature</a>
  </p>
</p>



<!-- TABLE OF CONTENTS -->
## Table of Contents

* [Architecture](#architecture)
  * [Built With](#built-with)
* [Template Overview](#template-overview)
  * [Core](#core)
    * [core-master](#core-master)
    * [01-kms](#01-kms)      
    * [01-newvpc](#01-newvpc)  
    * [02-securitygroups](#02-securitygroups) 
    * [03-rds](#03-rds)     
    * [04-apigateway](#04-apigateway) 
    * [04-route53](#04-route53) 
    * [04-elasticsearch](#04-elasticsearch)  
    * [04-s3](#04-s3) 
    * [04-s3-logs](#04-s3-logs) 
    * [04-sqs](#04-sqs)   
    * [04-sqsfifo](#04-sqsfifo)       
    * [05-cloudtrail](05-cloudtrail)
    * [05-dashboard](05-dashboard)
    * [05-guardduty](05-guardduty)
    * [05-loggroups](05-loggroups)
  * [Services](#services)
    * [service-master](#service-master)
    * [service-nested-master](#service-nested-master)    
    * [01-publicalb](#01-publicalb)      
    * [02-service-securitygroups](#02-securitygroups)  
    * [03-apigateway](#03-apigateway) 
    * [04-service](#04-service)     
  
* [Deployment](#deployment)
  * [Steps](#steps)
  * [Script Inputs](#script-inputs)
* [Roadmap](#roadmap)
* [Contributing](#contributing)



<!-- ABOUT THE PROJECT -->
## Architecture

<img src="https://github.com/CreditSaisonIndia/oneaboveall/blob/CLS-6162/CS-Assignment-Arch-Diagram.png" alt="Logo">

Oneaboveall contains nested stack templates require to automate the deployment of core & all microservices of Credit Saison India on AWS Cloudfomration Service.

### Built With
Oneaboveall is built using following tech stack
* [YAML](https://yaml.org/)
* [Bash](https://www.gnu.org/software/bash/)



<!-- TEMPLATE OVERVIEW -->
## Template Overview

To make the setup re-useable and easy to manage, CloudFormation and nested stacks have been created.  
The Templates are a logical grouping of services.

### Core

> Core is a parent stack for all independent aws core services templates.

- #### core-master
    - Placeholder for all parameters, definition and link to children templates.
    
- #### 01-kms
    - Template for AWS Key Management Service (KMS) which makes it easy for us to create and manage cryptographic keys and control their use across a wide range of AWS services and in the applications. 
    - AWS KMS is a secure and resilient service that uses hardware security modules that have been validated under FIPS 140-2, or are in the process of being validated, to protect all the keys.
    
- #### 01-newvpc
    - Creates a new VPC with 6 subnets spread across 2 Avaliablity Zones (The bash script will only take the 'a' and 'b' zones no matter which region you choose.
    - To allow for flow of traffic across subnets, it also creates the route tables (Seperate for each layer of the stack [Public/Web/Data] )
    - NATGW is are spun up with route setup to allow for instances in the private subnet 0.0.0.0/0 out access
    - IGWs are also associated with the VPC
    - Flowlogs are enabled and IAM Roles associated to the VPC service to allow the logs to be sent to CloudWatch

- #### 02-securitygroups
    - Creates the groups for each layer of the stack and nests the groups top to bottom
    - Public has 80 open to 0.0.0.0/0, Web is only open to Public over 80, and Data is only open to Web over 3306
    - This is an example of how to list things you need to use the software and how to install them.

- #### 03-rds
    - Creates an RDS Aroura Cluster with 2 nodes, 1 writer and 1 reader
    
- #### 04-apigateway
    - Creates two api gateways for internal proxy & partner proxy which acts as a reverse proxy to all our microservices. 
    
- #### 04-elasticsearch
    - Creates an elasticsearch cluster & its domain 

- #### 04-route53
    - Create a KSF Hosted Zone. It is effectively connecting user requests to infrastructure running in AWS. 

- #### 04-s3
    - Generic template use to create a s3 bucket with encryption enabled.
    
- #### 04-s3-logs
    - Generic template use to create a s3 bucket for logs purposes with encryption enabled.
    
- #### 04-sqs
    - Generic template use to create an amazon queue along with its dead letter queue configured.    
    
- #### 04-sqs-fifo
    - Generic template use to create an amazon fifo queue along with its dead letter queue configured.
    
- #### 05-cloudtrail
    - Enables cloudtrail service to log, continuously monitor, and retain account activity related to actions across AWS infrastructure of KSF.
    
- #### 05-dashboard
    - Creates a dashboard with a few metrices about the LoadBalancer and RDS within CloudWatch.
    - (*) Scope limits mentioned at the end
    
- #### 05-guardduty
    - Creates a guardduty detector to continuously monitors for malicious activity and unauthorized behavior to protect our AWS accounts, workloads, and data stored in Amazon S3.

- #### 05-loggroups
    - Generic template to create a log group inside cloudwatch service.
    

### Services

> Service is a parent stack for all of our microservices aws infrastrcuture templates.

- #### service-master
    - This is the parent stack containing nested stack of each of our microservice master child template so as to provide isolation.
    
- #### service-nested-master
    - This is the parent stack containing nested stack of each of the components used in building microservice infrastructure.
    
- #### 01-publicalb
    - Creates a Application loadbalancer and its listener groups.
    
- #### 02-securitygroups
    - Creates the groups for each layer of the stack and nests the groups top to bottom for a microservice.
    - Public has 80 open to 0.0.0.0/0, Web is only open to Public over 80, and Data is only open to Web over 3306
    
- #### 03-apigateway
    - Creates REST API & method for internal proxy gateway to route directs to a particular microservice.
    
- #### 04-service
    - Creates ec2, autoscaling group & configuration setup require to stand one microservice. 



## Deployment

### Steps

1. Run ghostrider to assign AWS Profile.

2. Run core script to deploy core infrastructure as follows

```sh
sh deployer-core.sh
```

3. Run service script to deploy all of our microservices infrastructure as follows

```sh
sh deployer-service.sh
```

### Script Inputs

INPUTS for the scripts will be as follows:
- **AWS Profile**                `Choose Default (as per ghostrider setup)`
- **Cloudformation StackName**   `core / service (as per your script)`
- **AWS Region**                 `us-east-1 / ap-south-1 / eu-west-1 (as per your account & region)`
- **AWS BucketName**             `cfn-templates-v2-{envtype} (envtype supported : {'dev' 'qa' 'qa2' 'uat' 'int' 'production'})`
- **Credit Saison Environment**  `{envtype} (envtype supported : {'dev' 'qa' 'qa2' 'uat' 'int' 'production'})`


<!-- ROADMAP -->
## Roadmap

See the [open issues](https://github.com/CreditSaisonIndia/oneaboveall/issues) for a list of proposed features (and known issues).



<!-- CONTRIBUTING -->
## Contributing

Contributions are what make oneaboveall such an amazing repo to learn, inspire, and create. Any contributions you make are **greatly appreciated**.

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

