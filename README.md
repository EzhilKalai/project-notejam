# Notejam on AKS and Azure SQL
 
## Previous Architecture and disadvantages
1.	Monolith application design
2.	It is a built-in webserver with SQLite DB on same machine
3.	Unable to update application components without downtime.
4.	Scalability is not possible as when required
5.	No resilience
6.	No environment for dev & test
7.	No monitoring is setup for metrics and logs

## Wanted solution:
1.	The Application must serve variable amount of traffic. Most users are active during business hours. During big events and conferences the traffic could be 4 times more than typical.
2.	Preserve data up to 3 years and recover it if needed.
3.	Continuity in service in case of datacenter failures.
4.	Should be region resilient.
5.	CI/CD without downtime.
6.	Spanning separate environments for dev\test\prod. 
7.	Infrastructure monitoring for quality assurance and security purposes.

## The Plan:
Use Notejam's Python/Django implementation
1.	Decouple the webserver and relational database
2.	Containerize the webserver and pushed to Azure ACR
3.	Replace SQLite to Azure managed AzureSQL
4.	The application source is continuously integrated, containerized and deployed on Azure ACR
5.	Databased requests are handled by AzureSQL
6.	The monitoring is done by Prometheus and visualized in Grafana

## Architecture overview
1.	AKS with RBAC for authentication, behind Azure Front Door to provide HA/DR.
2.	Istio for ingress controller.
3.	Kube hunter and Kube-bench to scan the cluster and report to DefectDojo. (Recomonded)
4.	DefectDojo with Azure AD authentication. (Recomonded)
5.	Istio for service mesh.
6.	Prometheus with for Metrics monitoring.
7.	Slack integration for alerts from Prometheus. (Recomonded)
8.	Elastic + Fluentd + Kibana for log monitoring or Azure OMS for both metrics and log monitoring. (Recommended)
9.	SonarQube hosted in the cluster - can be used by AppDev repos to publish static code analysis. (Recommended)

## Advantages on moving to Azure cloud
### Scalability & Elasticity
Because of the fact that the redesigned web tier of the application is now stateless, we can leverage scaling it horizontally. The containerized application is running on full-managed AKS making it de-facto serverless. AKS will automatically scale the number of running containers in and out based on the number of inbound web requests. This allows for cost-effective scaling: the application will scale in slowly to zero when there is no demand and it will scale out quickly to virtually limitless scale as soon as requests come in.

Database is now hosted on fully managed, scalable MySQL database which is easy to setup, operate and scale and also has zone redundant high availability. Intelligent performance recommendations providing custom analysis and suggestions for MySQL database optimization.

### Security & Compliance
1.	Discover, track, and remediate potential threats while they occur, with Advanced Threat Protection.
2.	Control data isolation by configuring a virtual network.
3.	Data is automatically encrypted at rest and in motion. Perform double encryption with custom keys and Transport Security Layer (TLS) 1.2 enforcement.
4.	Get industry-leading compliance, such as HIPAA, PCI DSS, FedRAMP, and ISO, built-in.
5.	Backup files are not user-exposed and cannot be exported. These backups can only be used for restore operations in Azure Database for MySQL. You can use mysqldump to copy a database.

To protect your customer data and application workloads in Azure Kubernetes Service (AKS), the security of the cluster is a key consideration. Kubernetes includes security components such as network policies and Secrets. Azure then adds in components such as network security groups and orchestrated cluster upgrades. These security components are combined to keep your AKS cluster running the latest OS security updates and Kubernetes releases, and with secure pod traffic and access to sensitive credentials.

### Development & Deployment
Follow a Blue-green deployment using Istio for the switch over and Jenkins for continues integration.

### Monitoring & Operations
Prometheus and Grafana are setup for the monitoring of AKS which is running the web tier of the application and custom dashboards are setup in Grafana to visualize and alerting.

### Environments & Stages
Separate namespaces are created for each environment (dev/test/prod) which enable you to deploy multiple environments with different configurations.

### Backups & Recovery
Azure Database for MySQL automatically creates server backups and stores them in user configured locally redundant or geo-redundant storage. Backups can be used to restore your server to a point-in-time. Backup and restore are an essential part of any business continuity strategy because they protect your data from accidental corruption or deletion.
