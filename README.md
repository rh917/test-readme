# H&E Public Web Environment - Craft 3 Build
<!-- PROJECT LOGO -->
<br />
<div align="center"><a href="https://he-equipment.com/"><img src="https://he.mediaframework.net/aws/pub-web/_assets/HE-Stacked-RGB-Rev.png" alt="HEES Logo" height="150"></a>

<h3 align="center">H&E AWS Craft 3 Deployment</h3>
<p align="center">Deploy Public Web Environment Infrastructure into AWS.</p>
</div>

<!-- ABOUT THE PROJECT -->
## About The Project

This repository contains terraform code and bootstrap execution scripts to deploy a full web stack on AWS. This solution is designed to deploy a holistic web delivery infrastructure on AWS for the H&E Public Web Environment.

These templates are deployable in two parts. The first template is the S3/IAM/CloudFront Resources (`~/tf-aws-heesweb/s3-resources/terraform/`) and the second template is the set of remaining infrastructure located in the root of the repository (`~/tf-aws-heesweb/`).

### Built With

* [AWS](https://aws.amazon.com/)
* [BASH](https://linuxconfig.org/bash-scripting-tutorial-for-beginners)
* [Terraform](https://www.terraform.io/intro)

|  | AWS Component | Use Case |
| -- | -- | -- |
|![S3](https://he.mediaframework.net/aws/pub-web/_assets/svc-logo/storage-cat.svg)| S3 | Static Content, logging destination |
| ![CloudFront](https://he.mediaframework.net/aws/pub-web/_assets/svc-logo/cdn-cat.svg) | CloudFront | Content delivery network for static content |
| ![IAM](https://he.mediaframework.net/aws/pub-web/_assets/svc-logo/iam-cat.svg) | IAM | Role based access control |
| ![Resource Groups](https://he.mediaframework.net/aws/pub-web/_assets/svc-logo/costmgmt-cat.svg) | Resource Groups | Resource grouping and tracking |
| ![VPC](https://he.mediaframework.net/aws/pub-web/_assets/svc-logo/vpc-svc.svg) | VPC, Route Tables, Subnets, Internet Gateways | Environment Networking |
| ![Prefix](https://he.mediaframework.net/aws/pub-web/_assets/svc-logo/vpc-svc.svg) | Prefix List | Predefined IP Prefix lists (for ACL's) |
| ![ALB](https://he.mediaframework.net/aws/pub-web/_assets/svc-logo/elb-svc.svg) | ALB/ELB | Application Load balancers |
| ![Security Groups](https://he.mediaframework.net/aws/pub-web/_assets/svc-logo/iam-cat.svg) | Security Groups                               | Traffic control between landscape components |
| ![EC2](https://he.mediaframework.net/aws/pub-web/_assets/svc-logo/compute-cat.svg) | EC2 | Web Servers |
| ![RDS](https://he.mediaframework.net/aws/pub-web/_assets/svc-logo/db-cat.svg) | RDS | AWS Aurora MySQL Backend |
| ![Route53](https://he.mediaframework.net/aws/pub-web/_assets/svc-logo/route53-svc.svg) | Route53 | DNS Resolution |
| ![ACM](https://he.mediaframework.net/aws/pub-web/_assets/svc-logo/acm-svc.svg) | ACM | Certificate Issuance (for cloudfront) |
|                                            |                                               |

<p align="right">(<a href="#top">back to top</a>)</p>

<!-- GETTING STARTED -->

## Prerequisites

The following components should be installed and configured before proceeding.

* [AWS Subscription](https://hees-inf.signin.aws.amazon.com/console)
* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
* [Terrform](https://www.terraform.io/downloads) installed
* [Linux Bash Shell](https://docs.microsoft.com/en-us/windows/wsl/install) w/ configured AWS credentials file
* [Git](https://git-scm.com/download/linux) is installed and configured

1. Install and configure AWS CLI (linux)

   ``` bash
   cd ~
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
   unzip awscliv2.zip
   sudo ./aws/install
   ```

2. Run AWS Configure to create and initially populate credentials file.

   ``` bash
   aws configure

   ## Enter the following information for your user
   AWS Access Key ID [None]: <your access key>
   <enter>
   AWS Secret Access Key [None]: <your access key secret>
   <enter>
   Default region name [None]: us-east-1
   <enter>
   Default output format [None]: json

   ## Run the following command to show the credentials file that was created
   cat ~/.aws/credentials
   ```

3. Modify the AWS credentials file for any additional environments. You'll notice that there is only one profile listed named `default` with your values. We're going to modify this file so it contains the various environments you have access to. Below, we define the H&E Sandbox, H&E Master Account, and H&E Infrastructure Accounts.

   Edit this file using the following command: `nano ~/.aws/cedentials`

   ``` bash
   [he-sandbox]
   aws_access_key_id = <your_key>
   aws_secret_access_key = <your_secret>
   region=us-east-1
   output=json

   [aws-master]
   aws_access_key_id = <your_key>
   aws_secret_access_key = <your_secret>
   region=us-east-1
   output=json

   [aws-infra]
   aws_access_key_id = <your_key>
   aws_secret_access_key = <your_secret>
   region=us-east-1
   output=json
   ```

4. Test that profiles and AWS S3 CLI are working with the following command. If the buckets are listed for the account, your access is setup appropriately.

   ```sh
   aws s3 ls --profile aws-infra
   ```

5. Install Terraform with the below commands

   ```sh
   sudo apt-get update && sudo apt-get install -y gnupg software-properties-common curl
   curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
   sudo apt-add-repository -y "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
   sudo apt-get update && sudo apt-get install terraform
   ```

   You can check the installation by running the following command (`terraform -v`). If terraform was successfully installed you should see the following message: 

   ```sh
   $ terraform -v
   Terraform v1.2.1
   on linux_amd64
   ```

6. Create a code working directory in your home folder: `mkdir ~/code`

<p align="right">(<a href="#top">back to top</a>)</p>

------
------

## Deployment Prep

1. Clone the repository to your local machine using your git client of choice or the CLI below. Then change directories into the s3 terraform directory

    ```bash
    cd ~/code
    git clone https://hees-inf@dev.azure.com/hees-inf/HEES-Craft-3/_git/tf-aws-heesweb
    cd ~/code/tf-aws-heesweb/s3-resources/terraform
    ```

2. Terraform [Workspaces](https://www.terraform.io/language/state/workspaces) can be used (and are in this project) to separate state files for different environments. This repository includes branches for both the `dev-env` and `env-prod` environments. 

   The command `terraform workspace list` will display a list of the workspaces that terraform is aware of. Run this command to see what is available.

   ```sh
   $ terraform workspace list
    * default
      dev
   ```

3. Ensure you're in the intended branch and proceed with selecting the appropriate workspace. In this case we will be using `dev`. Select the appropriate workspace below. 

   ```sh
   $ terraform workspace select dev
      Switched to workspace "dev".
   ```

4. Run the command `terraform init` to initialize the plugins and backend providers.

   ```sh
   terraform init
   ```

5. In this deployment we use `*.tfvars` files to define all of the environment-specific variables so that these files can be excluded from the repository for information governance. Ensure the parameters in the following files are appropriately filled out prior to deploying. 

   **_Failure to do this could result in overwriting or destroying an already-deployed landscape!_**

    ```sh
   ~/code/tf-aws-heesweb/terraform.tfvars
   ~/code/tf-aws-heesweb/s3-resources/terraform/terraform.tfvars
    ```

<p align="right">(<a href="#top">back to top</a>)</p>

## Deployment of Resources

### S3 Resources

1. After verifying the parameters in the `*.tfvars` files, proceed with deploying the S3 resources.

   ``` sh
   cd ~/code/tf-aws-heesweb/s3-resources/terraform
   
   # Confirm you're still in the appropriate workspace. If the anticipated workspace is not selected, refer to the previous section step 3 to select.

   terraform workspace list
   ```

2. The command `terraform plan` will do a dry-run of the deployment and highlight any errors. Run this command to view the resources that will be added/removed/changed.

   ```sh
   terraform plan
   ```

3. If satisfied with the deployment plan, proceed with the deployment using the following command

   ```sh
   terraform apply --auto-approve
   ```

<p align="right">(<a href="#top">back to top</a>)</p>

### Web Infrastructure Components

1. Now that the S3/CloudFront/IAM resources have been deployed, it's time to deploy the remaining web infrastructure. To do this, change directories to the root of the repository and ensure that you're in the appropriate workspace.

   ```sh
   $ cd ~/code/tf-aws-heesweb
   $ terraform workspace list
   $ terraform workspace list
     default
   * dev

   $ terraform init
   ```

2. With the terraform application initialized, run another `terraform plan` to verify output.

   ```sh
   terraform plan
   ```

3. Once validating the output, proceed with the deployment of the remaining resources.

   ```sh
   terraform apply --auto-approve
   ```

<!-- CONTACT -->
## Contact

* Ryan Hambacher - [Link](https://dot.cards/ryanhambacher?a=5qyv37y192kf575km937k01gm80ro5fw)
* __Project Link:__ [https://dev.azure.com/hees-inf/Azure_F5_Deployment](https://dev.azure.com/hees-inf/Azure_F5_Deployment)

<p align="right">(<a href="#top">back to top</a>)</p>
