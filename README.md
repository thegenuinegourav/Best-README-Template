<!-- PROJECT LOGO -->
<br />
<p align="center">
  <a href="https://github.com/CreditSaisonIndia/azkaban/blob/uat/README.md">
    <img src="images/logo.png" alt="Logo" width="80" height="80">
  </a>

  <h3 align="center">AZKABAN</h3>

  <p align="center">
    Credit Saison onion layer for partner ingress & egress interaction with core lending system.
    <br />
    <a href="https://github.com/CreditSaisonIndia/azkaban/blob/uat/README.md"><strong>Explore the docs »</strong></a>
    <br />
    <br />
    <a href="https://github.com/CreditSaisonIndia/azkaban/issues">Report Bug</a>
    ·
    <a href="https://github.com/CreditSaisonIndia/azkaban/issues">Request Feature</a>
  </p>
</p>



<!-- TABLE OF CONTENTS -->
## Table of Contents

* [Architecture](#architecture)
  * [Built With](#built-with)
* [Template Overview](#template-overview)
  * [master](#master)
  * [Lambda Layers](#lambda-layers)
    * [HMAC Layer](#hmac-layer)  
    * [Validation Layer](#validation-layer)  
  * [Asynchronous Process](#asynchronous-process)
    * [Post Lambda](#post-lambda)  
    * [Queue](#queue)      
    * [Consumer Lambda](#consumer-lambda)  
  * [Synchronous Process](#synchronous-process)
    * [Lambda](#lambda)  
  * [Global Variables](#global-variables)
    * [System Manager Paramters](#system-manager-parameters)  
    * [Secret Manager Keys](#secret-manager-keys)           
* [Deployment](#deployment)
  * [Steps](#steps)
  * [Script Inputs](#script-inputs)
* [Roadmap](#roadmap)
* [Contributing](#contributing)



<!-- ABOUT THE PROJECT -->
## Architecture

<img src="https://github.com/CreditSaisonIndia/azkaban/onionlayer.png" alt="Logo">

> Azkaban contains nested stack templates require to automate the deployment of onion layer of Credit Saison India on AWS Cloudfomration Service.
> It is our ingress & egress system for all of our core lending partners.
> This system acts as a wrapper around our core lending system.

### Built With
Azkaban is built using following tech stack
* [YAML](https://yaml.org/)
* [Bash](https://www.gnu.org/software/bash/)
* [Python](https://www.python.org/)
* [SAM](https://aws.amazon.com/serverless/sam/)
* [Python Cerberus](https://docs.python-cerberus.org/en/stable/)

<!-- TEMPLATE OVERVIEW -->
## Template Overview

To make the setup re-useable and easy to manage, CloudFormation and nested stacks have been created.  
The Templates are a logical grouping of services.

### master

> Placeholder for all parameters, definition and link to children templates.

### Lambda Layers

> Lambda Layers acts as a library package that can be included in lambda function for providing same functionality across all our lambda to avoid code redundancy. 

- #### HMAC Layer
    - This layer is verifying the integrity and authenticity of a partner request message.
    - This also provides secure mechanism for man-in-the-middle attack.
    - It calculates a message authentication code involving a hash function in combination with a secret key.
    
- #### Validation Layer
    - This layer is ensuring all the partner is well-structured & as per KSF data fields validation norms.
    - It uses python cerberus library as a powerful yet simple and lightweight data validation functionality out of the box.

### Asynchronous Process

> Partner Requests are asynchronous. Opt for this when incoming data is much larger & there should be a retry mechanism to post data into CLS if server is down.

- #### Post Lambda
    - This lambda will post all the incoming data to a queue.
    - Template & Function is built using SAM template.
    
- #### Queue
    - AWS SQS Queue to hold partner data.  
    - This will trigger Consumer Lambda as soon as data hits in it.
    - Deadletter queue is configured to hit missed data again to same consumer lambda.

- #### Consumer Lambda
    - Lambda which will take message from queue & hit the respective ksf microservice. 
    - After 5 retries, it pushes that message back to DeadLetter queue.    
    - Template & Function is built using SAM template.

### Synchronous Process

> Partner Requests are synchronous. Opt for this when you want to send immediate results to partner. 

- #### Lambda
    - Lambda to hit respective ksf microservice after data validation & computation.
    
### Global Variables

> Azkaban uses AWS environment global variables as an input to lambdas for hitting our ksf microservices.

- #### System Manager Parameters
    - Using ssm parameters as an input for base urls of our microservices.
    - Uses Transform function as a macro for fetching latest version of a parameter.
    - Macro: `GetSSMParameter`
    
- #### Secret Manager Keys
    - Using secret keys as an input for basic auth or api keys of our microservices.
    - `{{resolve:secretsmanager:BasicAuth-Secrets:SecretString:username}}`

## Deployment

### Steps

1. Run ghostrider to assign AWS Profile.

2. Run `deploy.sh` script to deploy onion layer infrastructure as follows

```sh
sh deploy.sh
```

### Script Inputs

INPUTS for the scripts will be as follows:
- **AWS Profile**                `Choose Default (as per ghostrider setup)`
- **Cloudformation StackName**   `onion`
- **AWS Region**                 `us-east-1 / ap-south-1 / eu-west-1 (as per your account & region)`
- **AWS BucketName**             `cfn-templates-v2-{envtype} (envtype supported : {'dev' 'qa' 'qa2' 'uat' 'int' 'production'})`
- **Credit Saison Environment**  `{envtype} (envtype supported : {'dev' 'qa' 'qa2' 'uat' 'int' 'production'})`


<!-- ROADMAP -->
## Roadmap

See the [open issues](https://github.com/CreditSaisonIndia/azkaban/issues) for a list of proposed features (and known issues).



<!-- CONTRIBUTING -->
## Contributing

Contributions are what make azkaban such an amazing repo to learn, inspire, and create. Any contributions you make are **greatly appreciated**.

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

