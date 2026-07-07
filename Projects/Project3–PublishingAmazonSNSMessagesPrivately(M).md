
# Project 3: Secure Healthcare Record Delivery via Private AWS SNS

## Problem Statement
XYZ Healthcare requires a highly secure platform to transmit sensitive patient reports. 
To meet compliance, traffic must not traverse the public internet. 
Implement a private messaging architecture by deploying an isolated VPC via CloudFormation, 
establishing an Interface VPC Endpoint for Amazon SNS, and hosting a backend application on an EC2 instance that 
publishes notifications privately.

Industry: Healthcare

Topics:
In this project, you will be working on a hospital project to send reports online and
develop a platform so the patients can access the reports via mobile and push
notifications. You will publish the report to an Amazon SNS keeping it secure and
private. Your message will be hosted on an EC2 instance within your Amazon
VPC. By publishing the messages privately, you can improve the message
delivery and receipt through Amazon SNS.

Highlights:
1. AWS CloudFormation to create a VPC
2. Connect VPC with AWS SNS
3. Publish message privately with SNS
   
---

## Steps To Solve

### Step 1: Deploy Infrastructure via CloudFormation
1. क्रिएट करा एक `template.yaml` फाईल:
   ```yaml
   Resources:
     MyVPC:
       Type: AWS::EC2::VPC
       Properties:
         CidrBlock: 10.0.0.0/16
     MyPrivateSubnet:
       Type: AWS::EC2::Subnet
       Properties:
         VpcId: !Ref MyVPC
         CidrBlock: 10.0.0.1/24

    ```

2. **CloudFormation Console** मध्ये जाऊन ही फाईल अपलोड करा आणि Stack क्रिएट करा.

### Step 2: Configure Private Connectivity (SNS Interface Endpoint)

1. **SNS Topic:** SNS Console मध्ये जाऊन एक 'Patient-Reports' Topic तयार करा.
2. **VPC Endpoint:** * **VPC Console > Endpoints > Create Endpoint**.
* **Service:** `com.amazonaws.[region].sns` निवडा.
* **Subnet:** स्टेप १ मध्ये तयार केलेली सबनेट निवडा.
* हे खाजगी कनेक्शन (Private Link) सुनिश्चित करते की तुमचा डेटा इंटरनेटवरून जाणार नाही.

### Step 3: Publish Messages Privately from EC2

1. **Launch EC2:** त्याच VPC च्या खाजगी सबनेटमध्ये एक `t2.micro` EC2 इन्स्टन्स लाँच करा.
2. **IAM Role:** EC2 ला एक IAM Role द्या ज्यामध्ये `AmazonSNSFullAccess` पॉलिसी असेल.
3. **Publishing Logic:** EC2 वरून खालीलप्रमाणे SNS मेसेज पाठवा:
```bash
aws sns publish \
  --topic-arn arn:aws:sns:region:account-id:Patient-Reports \
  --message "तुमचा हेल्थ रिपोर्ट तयार आहे. कृपया पोर्टलवर लॉग-इन करा." \
  --region [region]

```
---
## Critical Cleanup Process (Cost Management)

हे प्रोजेक्ट हाय-सिक्युरिटी नेटवर्कवर आधारित आहे, त्यामुळे खर्च टाळण्यासाठी खालीलप्रमाणे डिलीट करा:

1. **Delete VPC Endpoint:** हे सर्वात जास्त बिल वाढवू शकते. **VPC Console > Endpoints** मध्ये जाऊन Endpoint निवडा आणि डिलीट करा.
2. **Delete CloudFormation Stack:** **CloudFormation Console** मध्ये जाऊन Stack निवडा आणि **Delete** करा. यामुळे VPC आणि सबनेट आपोआप डिलीट होतील.
3. **Terminate EC2 & Delete SNS Topic:** EC2 इन्स्टन्स टर्मिनेट करा आणि SNS Topic डिलीट करा.
