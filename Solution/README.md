### Architecture diagram
![image](./../aws.drawio.png)

### Summary of Architecture components

| S/N | Category                     | Components                                               | Purpose / Function                                                   | Potential Challenges                      | Potential Solutions                      | Comments|
|-----|------------------------------|----------------------------------------------------------|-----------------------------------------------------------------------|-------------------------------------------|------------------------------------------|-|
| 1   | **Cloud Platform**           | AWS                                                      | Main platform for deploying the MIS                                  | Vendor lock-in                            |  Use multi-cloud strategy or abstraction layers (e.g., Terraform) to reduce dependency on a single provider | Cloud agnostic solution is viable but will be difficult to maintain if for e.g. terraform scripts syntax differs across cloud providers
| 2   | **Scalability & Elasticity** | Elastic Load Balancer (ELB), KEDA                        | Distributes traffic and enables auto-scaling                         |  Scaling complex microservices efficiently |  Set up scaling policies and use KEDA for event-based scaling within Kubernetes |
| 3   | **High Availability & Fault Tolerance** | Multi-AZ Setup, EKS                   | Provides redundancy and ensures continuous availability              |  Cross-AZ latency or network failure       |  Deploy resources across multiple AZs with automated failover configurations |
| 4   | **Low Latency & Global Delivery** | Amazon CloudFront, AWS Global Accelerator | Caches and accelerates content delivery globally                     |  Latency for regions without edge locations |  Use AWS Global Accelerator to optimize routing and place caching servers closer to users | I have not used AWS Global Accelerator before
| 5   | **Security & Compliance**    | AWS IAM, AWS KMS                                         | Manages access, security, and encryption                             |  Managing permissions and secure key storage |  Define least-privilege IAM policies, enforce multi-factor authentication, and use KMS for key rotation | 
| 6   | **Data Lake & Analytics**    | Amazon S3, Amazon RDS, Amazon DynamoDB                   | Stores structured and unstructured data                              |  Cost management and data retrieval times  |  Use data lifecycle policies for S3, use reserved instances for RDS, and DynamoDB's on-demand scaling |
| 7   | **CI/CD**                    | Git, AWS Elastic Container Registry (ECR), ArgoCD, GitHub Actions/GitLab CI | Manages source code, image storage, and deployment pipelines |  Versioning and ensuring smooth deployment |  Use branch-based deployment workflows, automated testing, and GitOps with ArgoCD for Kubernetes |Ideally the code is able to be built once and deployed everywhere (BODE) across environments
| 8   | **Cost Optimization**        | AWS Cost Explorer                                        | Monitors and optimizes cloud spending                                |  Unexpected cost increases                 |  Set budget alerts, review Cost Explorer reports, and right-size instances based on usage patterns | I have not used AWS Cost explorer before
| 9   | **Monitoring & Logging**     | Amazon CloudWatch, Prometheus, Grafana                   | Tracks metrics, application performance, and visualizes data         |  Maintaining visibility across distributed systems |  Set up alerts in CloudWatch and configure Grafana dashboards for Prometheus data |I have not used Grafana and Prometheus
||

Example workflow for developers can be:
1. Push code into Source Code Management tool, 
2. Their code will be built via github actions, aka an agent will clone the repository and run the dockerfile to build the image 
3. The image will then be pushed into ECR
4. An automation script can be used to modify the kubernetes manifest files (yaml) to update the version of the image being pulled from the ECR to the latest
5. ArgoCD will poll the SCM repository and detect the change in the version and deploy the latest kubernetes manifest yaml file 

Example workflow for reporters:
1. Login to MIS frontend UI
2. Run required service
    - Upload content (assuming code is built and network is set up to read and update s3 bucket/databases)
    - Run webscraper (assuming code is built and deployed on k8s cluster)
    - Run translator (assuming code is built and deployed on k8s cluster) 