A proper deployment of a GitHub Enterprise Server, including the implementation of all associated tasks, would be highly time-consuming. Therefore, we decided against proceeding with the implementation in order to focus on completing the other work packages.

# 1. Primary and Replika Management
An individual implementation of a status check and failover automatization is usually not needed, because the HA mode implements all these functions. 

In HA mode, data replication occurs in real-time between the primary and secondary nodes, using a shared external database and storage backend. The primary node continuously updates the secondaries to ensure data consistency, enabling seamless failover when necessary. GitHub Enterprise Server uses a highly available, external storage solution (like NFS or object storage) and an external database (e.g., PostgreSQL) to facilitate this.

The failover process in GitHub Enterprise Server HA mode is designed to occur automatically, ensuring that services remain available without manual intervention. Here’s how it works:
1.	Health Monitoring: GitHub Enterprise Server continuously monitors the health and availability of the primary instance through built-in health checks. It checks for signs of failure, such as loss of connectivity, hardware issues, or application errors.
2.	Automatic Detection of Failure: If the primary instance becomes unavailable or unhealthy, the HA setup automatically detects the failure condition.
3.	Automatic Failover: Upon detecting a failure, the HA system triggers an automatic failover. The process involves:
    - Promoting a Secondary: One of the secondary nodes is automatically promoted to become the new primary instance. The promotion is based on predefined rules, such as priority settings or proximity to the original primary.
    - Re-routing Traffic: Network configurations are adjusted automatically to reroute traffic to the newly promoted primary. This ensures a seamless transition for end-users with minimal downtime.
    - Synchronization Continuation: The remaining nodes are resynchronized to reflect the latest state of the new primary, ensuring data consistency across all instances.

# 2. User Alerting System
To create an alert when a user adds a private email address in GitHub Enterprise Server, we can leverage the platform's audit log events. These audit logs can be streamed and, within an Azure-based environment, captured using Azure Event Hub. Once the audit log events are ingested into the Event Hub, custom logic can be implemented to define actions for specific events.

For example, a serverless Azure Function can be used to process these events and log them into a Log Analytics Workspace. From there, you can utilize Azure's standard alerting and monitoring capabilities, such as Application Insights and Azure Alerts, to set up notifications and monitor the desired activities effectively.

![](/audit_log_stream.drawio.png)

# 3.
The deployment of the GitHub Enterprise Server should be standardized and fully automated, utilizing pipelines, scripts, and Infrastructure as Code (IaC). This approach ensures that new test environments can be provisioned at any time with minimal effort.

Once deployed, these test environments can be utilized with a variety of standard testing tools and frameworks. One excellent choice for test automation is Playwright. This framework allows for comprehensive testing of web applications across multiple browsers, providing robust support for asynchronous operations. Playwright not only enables the automation of test cases but also allows for the reproduction of test steps consistently, enhancing the reliability of the testing process. Additionally, it offers features like parallel execution and built-in support for debugging, making it a versatile tool in any testing toolkit.


# 4.

# 5. 