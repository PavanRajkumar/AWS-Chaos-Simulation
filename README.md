# Chaos Simulation on a website hosted on AWS
##### These are the steps to host a simple "Fact of the Day" website on an EC2 instance and attack it using the Gremlin contained chaos service! We'll use Amazon EC2, Cognito, IAM, and DynamoDB services. You'll need AWS free-tier and Gremlin trial accounts.

### Steps:

1. First you'll need to create a new user and give it administrator access on your AWS account.
2. Login to AWS CLI using your new user keys and create the DynamoDB table schema using the below commands
    ```
    aws dynamodb create-table --table-name facts --attribute-definitions \
    AttributeName=fact_id,AttributeType=N --key-schema \
    AttributeName=fact_id,KeyType=HASH \
    --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
    ```
3. Next populate the facts table you just created using the items.json file, remember you need to be in the same directory. Use the below command to do this.
    ```
    aws dynamodb batch-write-item --request-items file://items.json
    ```
4. Double check on your AWS console if the table has been created.
5. Next, create your EC2 instance, typically a low powered version, like a t2-micro so that we can easily max it out. Add the following bootstrap script while creating it to install and start httpd (Apache server). Don't forget to add HTTP security group and launch. Your EC2 instance is now up and running!
    ```bash
    #!/bin/bash
    yum update -y
    yum install -y httpd
    systemctl start httpd
    systemctl enable httpd
    ```

6. Using the CLI, create new identity pool, named DynamoPool, allow unauthenticated entities.
    ```
    aws cognito-identity create-identity-pool \
    --identity-pool-name DynamoPool \
    --allow-unauthenticated-identities \
    --output json
    ```
       
7. Create an IAM role named Cognito_DynamoPoolUnauth. 
    ```
    aws iam create-role --role-name Cognito_DynamoPoolUnauth --assume-role-policy-document file://myCognitoPolicy.json --output json
    ```
 
8. Grant the Cognito_DynamoPoolUnauth role read access to DynamoDB by attaching a managed policy (AmazonDynamoDBReadOnlyAccess).
    ```
    aws iam attach-role-policy --policy-arn arn:aws:iam::aws:policy/AmazonDynamoDBReadOnlyAccess --role-name Cognito_DynamoPoolUnauth ```

9. Get the IAM role Amazon Resource Name (ARN).
    ```
    aws iam get-role --role-name Cognito_DynamoPoolUnauth --output json 
    ```


10. Add our role to the Cognito Identity Pool. Replace the pool ID with your own pool ID and use the role ARN from the previous step.
    ```
    aws cognito-identity set-identity-pool-roles \
    --identity-pool-id "us-east-1:87a7206a-2a85-4171-bb6d-cd6208dd9d00" \
    --roles unauthenticated=arn:aws:iam::xxxxx:role/Cognito_DynamoPoolUnauthRole --output json
    ```


11. Double check it worked using: 
    ```
    aws cognito-identity get-identity-pool-roles  --identity-pool-id "us-east-1:87a7206a-2a85-4171-bb6d-cd6208dd9d00"
    ```
    
12. We can now specify the Cognito credentials in our application - i.e. in the JavaScript section of our webpage! Replace the identity pool ID with your own and the role ARN with your own role ARN. 
We are going to add this snippet to our index.html:

    ```
    AWS.config.credentials = new AWS.CognitoIdentityCredentials({
    IdentityPoolId: "us-east-1:87a7206a-2a85-4171-bb6d-cd6208dd9d00",
    RoleArn: "arn:aws:iam::xxxxx:role/Cognito_DynamoPoolUnauthRole"
    });
    ```
    
13. Now install and use Gremlin on your EC2 instance to destroy it! Look it up if you don't know how to use Gremlin.

14. Extraneous point for all those superstitious folks out there.