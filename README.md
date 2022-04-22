# Deploy a Fintech App on AWS using Plaid API Project 7

The word Fintech is an acronym for Financial Technology and it is used to describe any technology targeted at improving and automating the delivery and use of financial services.

A Fintech app helps companies, business owners and computers improve financial operations efficiency and processes through the use of specialized softwares and algorithms.

Examples of tasks made easier by the use of Fintech apps include check deposit via phone, credit request without visiting a brick and mortar branch, online money transfer. All these can be done via the use of smartphones.

This is an interesting project to me as I've spent majority of my working years in financial institutions and while I have interacted with different Fintech apps, this is the first time I will be deploying one.

## Benefits of Fintech Apps
- Viewing balances across multiple bank accounts.
- Initiating payments to friends.
- Applying for loans without gathering and scanning bank and income statements.
- Paying for things online using a “Buy Now Pay Later” plan.
- Showing monthly income and expense categories to help set budgets.
- Displaying overall investment performance across multiple brokerage accounts.
- Buying crypto-assets.

In this project, we will learn how to build and deploy a basic fintech app on Amazon Web Services (AWS) in under an hour by using the **[Plaid Link API](https://plaid.com/docs/link/)**. This app allows users to sign up, log in, select their bank from a list, connect to that bank, and display the latest transactions.

## About Plaid

**[Plaid](https://plaid.com/)** is a financial services company and [AWS Partner](https://partners.amazonaws.com/partners/0010h00001cBFNCAA4/Plaid) that helps fintech providers connect users safely to their bank accounts.

The [Plaid Link](https://plaid.com/docs/link/) acts as a secure proxy between a fintech app and a bank. With Plaid, application developers no longer need to worry about implementing scores of different ways to access data in myriad financial institutions.

Plaid is currently able to connect to more than 12,000 banks and financial institutions throughout the world. It provides a single API to connect to them. Currently, about 5,500 fintech apps use Plaid’s API to enable their users to access their bank accounts.

## What We Will Build in This Project

We will build a demo fintech app on AWS using the [AWS Amplify](https://aws.amazon.com/amplify/) framework and Plaid Link. AWS Amplify helps us quickly build a serverless web app with a React frontend, user sign-up and sign-in using [Amazon Cognito](https://aws.amazon.com/cognito/), an [Amazon API Gateway](https://aws.amazon.com/api-gateway/)-based REST API, and an [Amazon DynamoDB](https://aws.amazon.com/dynamodb/) database for storage.

AWS Amplify generates the code for signing up and authenticating users who are then stored in a [Cognito user pool](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools.html). It also helps create a REST API invoked by the React frontend and implemented by an [AWS Lambda](https://aws.amazon.com/lambda/) function behind Amazon API Gateway. The backend Lambda function sets up the Plaid Link which allows the end user to interact with a selected bank.

AWS Amplify also helps store the Plaid API key securely in [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/) so that it never needs to appear in the code or in a file. Plaid access tokens (described in the next section) are stored in the DynamoDB database.

This is a completely scalable and secure architecture which does not require the user to manage any server instances.

## How Plaid Link Works

To build an app using Plaid Link, you first need to go to [Plaid.com](https://plaid.com/), click on the **Get API Keys** button, and create an account. You can create a free sandbox account to start.

You can then log into your dashboard and find your sandbox API key under the menu for **[Team Settings – Keys](https://dashboard.plaid.com/team/keys)**.

The following diagram shows what our demo Web app needs to implement.

![](/images7/Plaid-workflow.png)

All API calls are made through a Plaid client object. The message flow is as follows:

1. The app first creates a Plaid client object by passing in the Plaid API key and Plaid client ID. It then calls the client’s **createLinkToken** method to obtain a temporary link token.
2. When the user selects a bank, the app uses the link token to open a Plaid Link to the bank and obtain a temporary public token.
3. The app then calls the client object’s **exchangePublicToken** method to exchange the public token for a permanent access token and an item ID that represents the bank.
4. The app stores the access token in DynamoDB for subsequent requests pertaining to that item. For example, the app can pass the access token to the client object’s **getTransactions** method to obtain a list of transactions within a specific date range.

## Building and Deploying the App

### Prerequisites

- Make sure you have created a sandbox account at Plaid as described above, and obtained your API keys.
- You also need to [install AWS Amplify](https://docs.amplify.aws/cli/start/install/).
- If you have not already done so, [create a default AWS configuration](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config) profile by running the **aws configure** command.

### Installing AWS Amplify
The preprequisites for installation is node v12.x or later and npm v6.x or later.

![](/images7/sw-version.png)

**Step 1**
Configure Amplify by running the following command:

```
curl -sL https://aws-amplify.github.io/amplify-cli/install | bash && $SHELL
```

![](/images7/amplify-cli.png)

**Step 2**
 Configure Amplify CLI on your local machine, so it can connect to your AWS account.

`$amplify configure`

![](/images7/AWSsignin.png)

This will direct you to the AWS sign in page to login in to your account. Once done, got back to the terminal and hit Enter to continue.

**Step 3**
Specify the AWS region and the intended amplify user id and you will be redirected to the AWS console to complete the creation.

![](/images7/amplify-user.png)

![](/images7/username.png)
![](/images7/permission.png)

**Step 4**
Review the settings and click create user. Once the user is created, you will be given an **access key ID** and a **secret access key**. It is a good idea to download the .csv file containing the access key details to a safe place for future reference because this file is made available only the first time when creating a new user.

**Step 5**
Go back to your terminal, press Enter to continue and you will be prompted to enter the access key details as shown below.

![](/images7/terminal.png)

### Building the App

Clone the repo and run *npm install*:

`$ git clone https://github.com/aws-samples/aws-plaid-demo-app.git`

![](/images7/git-clone.png)

Change directory to aws-plaid-demo-app

`$ cd aws-plaid-demo-app`

Then run:
`$ npm install`

![](/images7/plaid-install.png)

Initialize a new Amplify project. Hit **Return** to accept the defaults.

```bash
$ amplify init
```
![](/images7/amplify-init.png)
![](/images7/amplify-init2.png)

Next, add authentication:

```bash
$ amplify add auth 

? Do you want to use the default authentication configuration? 
>	Default configuration 
? How do you want users to be able to sign in? (Use arrow keys and space bar to select)
•	Username
? Do you want to configure advanced settings? 
>	No, I am done
```

![](/images7/add-auth.png)

Add the API:

```bash
$ amplify add api

? Please select from one of the below mentioned services: REST
? Provide a friendly name for your resource to be used as a label for this category in the project: plaidtestapi
? Provide a path (e.g., /book/{isbn}): /v1
? Choose a Lambda source: Create a new Lambda function
? Provide an AWS Lambda function name: plaidaws
? Choose the runtime that you want to use: NodeJS
? Choose the function template that you want to use: Serverless ExpressJS function (Integration with API Gateway)
? Do you want to configure advanced settings? Yes
? Do you want to access other resources in this project from your Lambda function? No
? Do you want to invoke this function on a recurring schedule? No
? Do you want to enable Lambda layers for this function? No
? Do you want to configure environment variables for this function? Yes
? Enter the environment variable name: CLIENT_ID
? Enter the environment variable value: [Enter your Plaid client ID]
? Select what you want to do with environment variables: Add new environment variable
? Select the environment variable name: TABLE_NAME
? Enter the environment variable value: plaidawsdb
? Select what you want to do with environment variables: I am done
? Do you want to configure secret values this function can access? Yes
? Enter a secret name (this is the key used to look up the secret value): PLAID_SECRET
? Enter the value for PLAID_SECRET: [Enter your Plaid sandbox API key - hidden]
? What do you want to do? I'm done
? Do you want to edit the local lambda function now? No
? Restrict API access: No
? Do you want to add another path? No
```

![](/images7/plaid-setup.png)
![](/images7/plaid-setup2.png)

Copy the Lambda source file, install dependencies, and push:

```bash
$ cp lambda/plaidaws/app.js amplify/backend/function/plaidaws/src/app.js
$ cd amplify/backend/function/plaidaws/src
$ npm i aws-sdk moment plaid@8.5.4
```
![](/images7/dependencies.png)

Next, push

`$ amplify push`

![](/images7/push.png)
![](/images7/push2.png)
![](/images7/push3.png)
![](/images7/push4.png)
![](/images7/push5.png)

Add a database:

```bash
$ amplify add storage

? Please select from one of the below mentioned services: NoSQL Database
? Please provide a friendly name for your resource that will be used to label this category in the project: plaidtestdb
? Please provide table name: plaidawsdb

You can now add columns to the table.

? What would you like to name this column: id
? Please choose the data type: string
? Would you like to add another column? Yes
? What would you like to name this column: token
? Please choose the data type: string
? Would you like to add another column? No
? Please choose partition key for the table: id
? Do you want to add a sort key to your table? No
? Do you want to add global secondary indexes to your table? No
? Do you want to add a Lambda Trigger for your Table? No
Successfully added resource plaidtestdb locally
```
![](/images7/storage.png)

Update the Lambda function to add permissions for the database:

```bash
$ amplify update function

? Select the Lambda function you want to update plaidaws
General information
- Name: plaidaws
- Runtime: nodejs

Resource access permission
- Not configured

Scheduled recurring invocation
- Not configured

Lambda layers
- Not configured

Environment variables:
- CLIENT_ID: plaidclientid

Secrets configuration
- PLAID_SECRET

? Which setting do you want to update? Resource access permissions
? Select the categories you want this function to have access to.
	storage
? Storage has 2 resources in this project. Select the one you would like your Lambda to access plaidawsdb
? Select the operations you want to permit on plaidawsdb: create, read, update, delete
? Do you want to edit the local lambda function now? No
```
![](/images7/update.png)

### Deploying the App

Add hosting for the app:

```bash
$ amplify add hosting

? Select the plugin module to execute:
>	Hosting with Amplify Console (Managed hosting)
? Choose a type
>	Manual deployment
```
![](/images7/add-hosting.png)

Deploy the app:

`$ amplify publish`

![](/images7/publish.png)

![](/images7/publish2.png)

### Testing the App

Go to the URL displayed by the *amplify publish* command, and sign up as a new user. After logging in, select a bank from the list displayed.

![](/images7/signup2.png)

![](/images7/plaid.png)

![](/images7/plaid-connect2.png)

![](/images7/bank2.png)

If you are using the sandbox environment, use the credentials **user_good / pass_good** to access the bank and display the transactions.

![](/images7/creds.png)

![](/images7/success.png)

N:B: Selecting Bank of America did not work as it required second factor authentication which I don't have access to, but Chase bank worked.

## Clean up
It's important to clean up and terminate the resources/services utilized in AWS otherwise our account might go in the red.

- Login to AWS console, type amplify in the search bar and select AWS Amplify from the search result

![](/images7/amplify-aws.png)

- Click on the awsplaiddemoapp that we created, actions, delete.

![](/images7/del-app.png)

- confirm delete
![](/images7/app-del.png)

## Conclusion

The walkthrough in this project demonstrates how easy it is to use AWS Amplify to create a secure, scalable, and completely serverless fintech app on AWS that allows users to sign up, select from among the 10,000 banks that Plaid Link connects to, and obtain the transaction history for a particular account.

From here, you can add features such as making payments to friends or vendors, displaying balances across multiple accounts, sending low balance alerts and helping set a budget.