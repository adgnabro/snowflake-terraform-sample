# Terraforming Snowflake

This tutorial demonstrates how you can use Terraform to manage your Snowflake configurations in an automated and source-controlled way. We show you how to install and use Terraform to create and manage your Snowflake environment, including how to create a database, schema, warehouse, multiple roles, and a service user. 

Terraform to integrate with Snowflake use-cases:

- Set up storage in your cloud provider and add it to Snowflake as an external stage
- Add storage and connect it to Snowpipe
- Create a service user and push the key into the secrets manager of your choice, or rotate keys

Prerequisites :
- Snowflake account
- GitHub account
- git command-line client
- Text editor of your choice
- Terraform installed

## Create a new repository

    mkdir snowflake-terraform-sample && cd snowflake-terraform-sample
    echo "# sfguide-terraform-sample" >> README.md
    git init
    git add README.md
    git commit -m "first commit"
    git branch -M main
    gh create snowflake-terraform-sample --public # Create a new public repository in Github
    git remote add origin git@github.com:YOUR_GITHUB_USERNAME/snowflake-terraform-sample.git
    git push -u origin main

## Create a service user for Terraform

We will now create a user account separate from your own that uses key-pair authentication. The reason this is required in this lab is due to the provider's limitations around caching credentials and the lack of support for 2FA. Service accounts and key pair are also how most CI/CD pipelines run Terraform.

### Create an RSA key for authentication

This creates the private and public keys we use to authenticate the service account we will use for Terraform.

    cd ~/.ssh
    openssl genrsa 2048 | openssl pkcs8 -topk8 -inform PEM -out snowflake_tf_snow_key.p8 -nocrypt
    openssl rsa -in snowflake_tf_snow_key.p8 -pubout -out snowflake_tf_snow_key.pub

### Create the user in Snowflake

Log in to the Snowflake console and create the user account by running the following command as the ACCOUNTADMIN role.

Display the content of the ~/.ssh/snowflake_tf_snow_key.pub :

    cat ~/.ssh/snowflake_tf_snow_key.pub 

- Copy the text contents starting after the BEGIN PUBLIC KEY header, and stopping just before the END PUBLIC KEY footer in the SQL query variable RSA_PUBLIC_KEY_HERE
- Execute both of the following SQL statements to create the User and grant it access to the SYSADMIN and SECURITYADMIN roles needed for account management

### Add account information to environment

Run these commands in your shell. Be sure to replace the YOUR_ACCOUNT_LOCATOR and YOUR_REGION_HERE placeholders with the correct values.

NOTE: Setting SNOWFLAKE_REGION is required if you are using a Legacy Account Locator.

    export SNOWFLAKE_USER="tf-snow"
    export SNOWFLAKE_AUTHENTICATOR=JWT
    export SNOWFLAKE_PRIVATE_KEY=`cat ~/.ssh/snowflake_tf_snow_key.p8`
    export SNOWFLAKE_ACCOUNT="ACCOUNT_LOCATOR"
    export SNOWFLAKE_REGION_ID="SNOWFLAKE_REGION"

If you plan on working on this or other projects in multiple shells, it may be convenient to put this in a snow.env file that you can source or put it in your .bashrc or .zshrc file. For this lab, we expect you to run future Terraform commands in a shell with those set.

## Preparing the project to run

To set up the project to run Terraform, you first need to initialize the project.

Run the following from a shell in your project folder:

    terraform init

## Commit changes to source control

Run the following to check in your files for future change tracking:

    git checkout -b dbwh
    git add main.tf
    git add .gitignore
    git commit -m "Add Database and Warehouse"
    git push origin HEAD

## Prepare your change

Your specific workflow will depend on your requirements, including your compliance needs, your other workflows, and your environment and account topology.

For this lab, you can simulate the proposed CI/CD job and do a plan to see what Terraform wants to change. During the Terraform plan, Terraform performs a diff calculation to compare the desired state with the previous/current state from the state file to determine the acitons that need to be done.

From a shell in your project folder (with your Account Information in environment), run:

    terraform plan

## Running Terraform

Now that you have reviewed the plan, we simulate the next step of the CI/CD job by applying those changes to your account.

- From a shell in your project folder (with your Account Information in environment) run:
    
    terraform apply

- Terraform regenerates the execution plan (unless you supply an optional path to the plan.out file) and applies the needed changes after confirmation. In this case, Terraform will create two new resources, and have no other changes.

- Log in to your Snowflake account and verify that Terraform created the database and the warehouse.

## Changing and Adding Resources

## Get the public and private key information for the application

## Commit Changes to Source Control

Let's check in our files for future change tracking:

    git checkout -b svcuser
    git add main.tf
    git add outputs.tf
    git commit -m "Add Service User, Schema, Grants"
    git push origin HEAD

## Apply and Verify the Changes

To simulate the CI/CD pipeline, we can apply the changes to conform the desired state with the stored state.

From a shell in your project folder (with your Account Information in environment) run:
    
    terraform apply

https://quickstarts.snowflake.com/guide/terraforming_snowflake