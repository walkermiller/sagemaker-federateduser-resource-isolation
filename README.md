## Summary
Introducing three new tags to be used with resources to control individual access. For users logged into the console, their access will be determined by the tag "user_email" matching the email associated with their user information that has been  synched with the IdP. 
For users logged into an instance of SageMaker Studio, their access will be determined by the tag "user_profile" matching the [SageMaker User Profile](https://docs.aws.amazon.com/sagemaker/latest/dg/domain-user-profile.html) defined. 

Since console, or cli users will be limited to the SageMaker User Profiles they are granted access to with the "username" tag, they will then be limited to the resources with the appropriate "userprofile" tag when logged into SageMaker. 


## In the SSO Account
Look up the InstanceArn for your SSO setup and export it below. 

```
export InstanceArn=
aws sso-admin update-instance-access-control-attribute-configuration --cli-input-json file://attribute-configuration.json --instance-arn $InstanceArn
aws sso-admin create-permission-set --name sagemaker-developer-test --instance-arn $InstanceArn
aws sso-admin put-inline-policy-to-permission-set  --$InstanceArn --permission-set-arn --cli-imput-json
```
**Note**  
The attribute configuration is setting two attributes that will be passed as attribute tags to the active session: user_name and user_department. The later is not used in the following examples, but can be used when you want to restrict access to anyone in a particular group. For instance, you may want to add another rule in the sagmaker-developer permission set that allows anyone within a department to access any secrets that are tagged ```user_deparment=${aws:PrincipalTag/user_email}```


## In the SageMaker Account
## Create Execution Role and Execution Policy
Two specifics to note:
1. The trust policy of the execution role must include permissions to sts:SetSourceIdentity
2. The execution policy should be modified to only allow acces where a userProfile tag matches ${aws:SourceIdentity}
For an existing domain
```
aws sagemaker list-domains
```
Pick the appropriate domain listed and set is as the sagemakerDomain below. Also, include the department Id of the group that will be utilizing the execution policy. 
The domain and the departmentId could be the same value potentially. 
```
export sagemakerDomain=
export departmentId=

aws iam create-role --role-name ${sagemakerDomain}SagemakerExecutionRole --assume-role-policy-document file://sagemaker-execution-role-trust-policy.json --tags "Key=user_department,Value=$departmentId"
policyArn=`aws iam create-policy --policy-name ${sageMakerDomain}SagemakerExecutionPolicy --policy-document file://sagemaker-execution-role-policy.json | jq -r .Policy.Arn`
aws iam attach-role-policy --role-name ${sagemakerDomain}SagemakerExecutionRole --policy-arn ${policyArn}
```

