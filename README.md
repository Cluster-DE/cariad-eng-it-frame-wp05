A proper deployment of a GitHub Enterprise Server, including the implementation of all associated tasks, would be highly time-consuming. Therefore, we decided against proceeding with the implementation in order to focus on completing the other work packages.

# 1. Primary and Replika Management
An individual implementation of a status check and failover automatization is usually not needed, because the HA mode implements all these functions. 

In [HA mode](https://docs.github.com/en/enterprise-server@3.10/admin/monitoring-and-managing-your-instance/configuring-high-availability/about-high-availability-configuration), data replication occurs in real-time between the primary and secondary nodes, using a shared external database and storage backend. The primary node continuously updates the secondaries to ensure data consistency, enabling seamless failover when necessary. GitHub Enterprise Server uses a highly available, external storage solution (like NFS or object storage) and an external database (e.g., PostgreSQL) to facilitate this.

The failover process in GitHub Enterprise Server HA mode is designed to occur automatically, ensuring that services remain available without manual intervention. Here’s how it works:
1.	Health Monitoring: GitHub Enterprise Server continuously monitors the health and availability of the primary instance through built-in health checks. It checks for signs of failure, such as loss of connectivity, hardware issues, or application errors.
2.	Automatic Detection of Failure: If the primary instance becomes unavailable or unhealthy, the HA setup automatically detects the failure condition.
3.	Automatic Failover: Upon detecting a failure, the HA system triggers an automatic failover. The process involves:
    - Promoting a Secondary: One of the secondary nodes is automatically promoted to become the new primary instance. The promotion is based on predefined rules, such as priority settings or proximity to the original primary.
    - Re-routing Traffic: Network configurations are adjusted automatically to reroute traffic to the newly promoted primary. This ensures a seamless transition for end-users with minimal downtime.
    - Synchronization Continuation: The remaining nodes are resynchronized to reflect the latest state of the new primary, ensuring data consistency across all instances.

# 2. User Alerting System
To create an alert when a user adds a private email address in GitHub Enterprise Server, we can leverage the platform's [audit log events](https://docs.github.com/en/enterprise-cloud@latest/admin/monitoring-activity-in-your-enterprise/reviewing-audit-logs-for-your-enterprise/audit-log-events-for-your-enterprise#user_email). These audit logs can be streamed and, within an Azure-based environment, captured using Azure Event Hub. Once the audit log events are ingested into the Event Hub, custom logic can be implemented to define actions for specific events.

For example, a serverless Azure Function can be used to process these events and log them into a Log Analytics Workspace. From there, you can utilize Azure's standard alerting and monitoring capabilities, such as Application Insights and Azure Alerts, to set up notifications and monitor the desired activities effectively.

![](/audit_log_stream.drawio.png)

# 3. Test Environment Setup
The deployment of the GitHub Enterprise Server should be standardized and fully automated, utilizing pipelines, scripts, and Infrastructure as Code (IaC). This approach ensures that new test environments can be provisioned at any time with minimal effort.

Once deployed, these test environments can be utilized with a variety of standard testing tools and frameworks. One excellent choice for test automation is Playwright. This framework allows for comprehensive testing of web applications across multiple browsers, providing robust support for asynchronous operations. Playwright not only enables the automation of test cases but also allows for the reproduction of test steps consistently, enhancing the reliability of the testing process. Additionally, it offers features like parallel execution and built-in support for debugging, making it a versatile tool in any testing toolkit.


# 4. Georeplica Setup
The georeplica setup of the GitHub Enterprise server should be setup according to the [documentation](https://docs.github.com/en/enterprise-server@3.10/admin/monitoring-and-managing-your-instance/configuring-high-availability/creating-a-high-availability-replica#creating-geo-replication-replicas). 

1. Create Virtual Machines:  
Provision multiple Azure VMs in different regions for the primary and secondary GitHub Enterprise Server instances.

2. Install and Configure GHES:  
Install GitHub Enterprise on each VM. Follow GitHub’s installation documentation.
Configure the primary instance first, then replicate settings to the secondary instance.

3. Database Configuration:  
Use an Azure SQL Database or Azure Managed Instance for HA database needs. Ensure both instances can access the database.
Set up database replication as per GitHub’s guidelines for HA configurations.

4. Data Replication:  
Use the ghes-georeplica command to configure geo-replication between the primary and secondary instances.
Set up a secure connection (VPN or Azure ExpressRoute) between regions for data replication.

5. Load Balancer:  
Configure Azure Load Balancer or Application Gateway to manage traffic between the instances and ensure failover capabilities.

# 5. Repository Restore
The ghe-restore command is part of the GitHub backup utils. Please use [this](https://github.com/github/backup-utils/blob/master/docs/README.md) documentation as a reference.

1. Prepare the Environment:  
Log into the GitHub Enterprise server where you want to restore the repository.
Ensure you have the necessary permissions to perform the restore operation.

2. Stop Services (if necessary):  
If you are restoring a live instance, it’s a good practice to stop the GitHub services to prevent data inconsistencies. Use:
```
ghe-service stop
```

3. Locate the Backup File:   
Identify the backup file you want to restore. Backup files are usually named with a timestamp and might look like ```ghe-backup-YYYY-MM-DD.tar.gz```.

4. Run the Restore Command:  
Use the ghe-restore command to initiate the restore process. The basic syntax is:
```
ghe-restore <path-to-backup-file>
```
Example:
```
ghe-restore /path/to/ghe-backup-YYYY-MM-DD.tar.gz
```

5. Start Services:  
If you stopped the GitHub services, restart them using:
```
ghe-service start
```
