# Serverless Lead Capture on AWS: The Epic Books
I built a system on AWS that allows businesses to collect customer names and email addresses through a website. The information is stored securely for future marketing campaigns, and customers receive downloadable content via email as a reward.


## 1. Project Overview
Epic Books is a mock company that needs a simple way to collect customer names and email addresses through their website. Older setups often relied on dedicated servers that require ongoing maintenance, manual updates, and management of insfratructure. These systems can also struggle when there is a lot of traffic and are also very expensive to operate even when usage is low. The business needed a solution that could reliably keep customer data, remain fast for global users, and securely store the data without requiring dedicated infrastructure.

To solve this, I built a **serverless solution on AWS**. This eliminates the need to manage servers, which reduces infrastructure costs and also eliminates the need to hire staff for maintenance (patching). The solution automatically scales as website traffic increases and provides stronger security because it uses AWS managed services instead of vulnerable self-hosted systems.
---

## 2. Business Requirements
Epic Books provided the following requirements:

- They need a static website built with HTML, CSS, and JavaScript.  
- Customers must be able to download an ebook. 
- Customers must receive an email notification after downloading the ebook. 
- Customer details (name and email) must be stored in a database after download.
- The website must serve global users and remain fast and responsive. 
- The website has 500 active users with 10–15 ebook downloads per day.  
- All traffic must use HTTPS  
- The domain name must be theepicbooks.com.

---

## 3. Well-Architected Framework Considerations
I designed the solution using the AWS Well-Architected Framework in mind. The following pillars are the ones that  I felt are most relevant to the business’s needs:

- **Security:** HTTPS, secure storage of customer data
- **Performance Efficiency:** The website should be fast for global users
- **Operational Excellence:** Email notifications, monitoring and logging
- **Cost Optimization:** Stick to serverless services as much as possible to reduce idle costs (Cloud services that only cost money when they’re actually being used)
- **Reliability:** The database should not lose data and should also scale automatically

**Prioritized pillars for design decisions:**

1. Security
2. Cost Optimization
3. Performance Efficiency

These pillars guided all my choices. It’s also worth mentioning that these priorities might be switched and modified based on business requirements.


---

## 4. Architecture Overview

### 4.1 Architecture Diagram
![Architecture Diagram](Serverless_Lead_Capture_Diagram.png) 


### 4.2 How It Works (Summary)
- Static website hosted on **Amazon S3**  
- Distributed globally via **Amazon CloudFront**  
- HTTPS provided by **AWS Certificate Manager**  
- Form submissions sent to **Amazon API Gateway**  
- All processing done by **AWS Lambda**  
- Customer data stored in **Amazon DynamoDB**  
- Confirmation emails sent via **Amazon SES**  
- **Amazon CloudWatch** monitors and logs all backend activity  
- **IAM Roles & Policies** enforce least-privilege access  

---

## 5. Design Decisions & Implementation

### 5.1 Hosting Static Website
**Solution:** Amazon S3 + Amazon CloudFront + AWS Certificate Manager  

**Why:**  
- Amazon S3 can host static websites and has high availability and is scalable
- Amazon CloudFront improves speed and enables HTTPS  
- AWS Certificate Manager ensures secure connections  

**Instructions:**  
1. Prepare website files and code (HTML, FONTS, CSS, JavaScript, images)
2. Create an S3 bucket for static website hosting.
3. Enable the static website feature on S3 bucket.
4. Configure public access and added a bucket policy that grants users "GetObject" permissions.
5. Upload an error document for when users are unable to access the site.
6. Upload all website files to the S3 bucket.
7. Create a CloudFront distribution for the website using the S3 bucket as the origin type.
8. Request a public TLS certificate in ACM for theepicbooks.com.
9. Route traffic to the CloudFront distribution on Route 53.


---

### 5.2 Contact Form / Lead Capture
**Solution:** For this section I used Amazon API Gateway, AWS Lambda, Amazon SES, IAM Roles & Policies and Amazon CloudWatch. 

**Why:**  
- API Gateway receives the form submission from the website and passes the data to the backend for processing
- AWS Lambda receives the form data from API Gateway, processes it, saves the lead information to DynamoDB, and sends a notification email through Amazon SES.
- Amazon SES sends reliable emails  
- IAM Roles & Policies enforce least-privilege permissions for Lambda  
- Amazon CloudWatch monitors logs and performance  

**Instructions:**  
1. Setup SES sender and receiver identities  
2. Create an IAM role with permissions for Lambda to send emails and write CloudWatch logs  
3. Create a Lambda function to process form submissions and send emails via SES  
4. Test the Lambda function  
5. Create a REST API in Amazon API Gateway for form submissions  
6. Enable CORS and deploy the API  
7. Test API with CURL (simulate browser requests)  
8. Update website form to point to API endpoint  
9. Invalidate CloudFront cache to serve updated files  

---

### 5.3 Data Storage (Customer information)
**Solution:** Amazon DynamoDB  

**Why:**  
- Scalable and reliable database for storing leads  
- Integrates easily with AWS Lambda  
- Fully managed, serverless  

**Instructions:**  
1. Create DynamoDB table  
2. Grant Lambda permissions to write to the table via IAM role  
3. Update Lambda function to store submissions  
4. Test end-to-end workflow  

**Ebook Delivery:**  
- Lambda triggers SES email with secure S3 link or pre-signed URL for ebook download  

---

## 6. Trade-offs & Alternatives Considered

### 6.1 Hosting Static Website
- **Direct S3 Access**  
  - Pros: simpler, immediate updates  
  - Cons: slower globally, HTTPS requires CloudFront, security risks  

- **AWS Global Accelerator**  
  - Pros: resilient routing  
  - Cons: no caching, higher cost, fixed fees  

### 6.2 Contact Form Backend
- **Application Load Balancer instead of API Gateway**  
  - Pros: potentially cheaper at high traffic  
  - Cons: more complex security and API management  

- **Lambda Function URL instead of API Gateway**  
  - Pros: simpler setup, lower cost  
  - Cons: fewer security options, no throttling  

- **Amazon EC2 instead of Lambda**  
  - Pros: good for long-running workloads  
  - Cons: requires server management, patching, scaling, and higher cost  

---

## 7. Reliability & Resilience
**Managed AWS services minimize single points of failure:**  

- **Amazon S3:** versioning + CloudFront caching  
- **Amazon CloudFront:** multi-edge redundancy, ACM auto-renewal  
- **Amazon API Gateway:** throttling, CloudWatch alarms  
- **AWS Lambda:** timeout/memory config, optional dead-letter queue  
- **Amazon DynamoDB:** multi-AZ, on-demand scaling, point-in-time recovery  
- **Amazon SES:** monitor quotas, sandbox removal  
- **Amazon CloudWatch:** log retention, alarms  

**Summary:** Core data (leads) is secure even if email delivery fails.

---

## 8. Performance & Scalability
- Amazon CloudFront caches content globally for faster delivery  
- AWS Lambda automatically scales with concurrent requests  
- Amazon DynamoDB on-demand scaling ensures low latency  
- Amazon SES scales for email delivery  
- Fully managed services allow seamless scaling without manual intervention  

---

## 9. Security Considerations
- HTTPS for all traffic (**AWS Certificate Manager**)  
- IAM Roles & Policies enforce least-privilege access  
- Web Application Firewall rules applied via Amazon CloudFront (blocking SQLi, XSS, bots)  
- S3 bucket policies + CloudFront Origin Access Control prevent unauthorized access  
- CloudWatch alerts for security events and misconfigurations  

---

## 10. Cost Considerations
- Serverless architecture reduces idle infrastructure costs  
- Pay-as-you-go services match usage patterns  
- Amazon CloudFront caching reduces S3 data transfer costs  
- Fully managed services reduce operational overhead  

---

## 11. Future Improvements
- Add analytics (Amazon Athena / QuickSight)  
- Advanced email personalization with SES templates  
- Multi-language website support  
- Additional Web Application Firewall rules for security  
- Lambda provisioned concurrency or DynamoDB adaptive capacity for high traffic  

---

## 12. Challenges & Lessons Learned
- AWS Certificate Manager requires domain ownership  
- Planning infrastructure while documenting at the same time is challenging  
- Understanding individual services is crucial; tutorials alone are not enough