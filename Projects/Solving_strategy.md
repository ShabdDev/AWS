### **Projects: Master Tracking Table**

| Project Name | Architecture Highlights | Credit/Free Tier Status | Action Required (Before 12 July Deadline) |
| --- | --- | --- | --- |
| **Project 1: Multi-Tier Website (EC2 + RDS + ASG)** | Web Tier (ASG) + Database Tier (RDS) | ❌ **NOT FREE / HIGH COST** | **तातडीने करा (Immediately):** यात ASG मुळे अनेक EC2 इन्स्टन्स आणि RDS बॅकएंडला सतत सुरू राहतात. प्रॅक्टिकल संपताच **Auto Scaling Group डिलीट करून सर्व EC2 आणि RDS Instance बंद करा.** |
| **Project 2: Website Orchestration (Beanstalk + RDS)** | Beanstalk (PaaS) + External RDS | ❌ **NOT FREE / HIGH COST** | **तातडीने करा (Immediately):** Elastic Beanstalk आपोआप Load Balancer आणि EC2 सुरू करतो. प्रॅक्टिकल संपताच **Beanstalk Environment Terminate करा आणि RDS डिलीट करा.** |
| **Project 3: Secure Healthcare SNS (VPC + Endpoint)** | Private VPC + VPC Endpoint + SNS | ❌ **HIGH COST (Endpoint)** | **तातडीने करा (Immediately):** **Interface VPC Endpoints** चे तासाचे चार्जेस जास्त असतात. प्रॅक्टिकल संपताच **VPC Endpoint आणि CloudFormation Stack त्वरित डिलीट करा.** |

---
