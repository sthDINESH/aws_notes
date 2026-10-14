# DocumentDB

## What?

- Amazon DocumentDB is a managed NoSQL database service for storing JSON-like documents.
- Think of it as a flexible filing cabinet: each document can have different fields without requiring a fixed table structure.
- Documents are grouped into collections, similar to how rows are grouped into tables in a relational database.
- DocumentDB is designed to be compatible with MongoDB applications and drivers, but it is not the same product and does not support every MongoDB feature.

## How?

- Create a DocumentDB cluster inside an Amazon VPC.
- Connect to the cluster using a MongoDB-compatible driver or tool.
- Store and query documents using a document-oriented data model.
- Use a primary instance for writes and optional replica instances for read scaling and high availability.
- Configure security groups to control which applications can connect to the database.
- AWS manages much of the infrastructure, including storage, backups, monitoring, and failover.

## Why?

- A flexible document structure is useful when application data changes frequently or does not fit neatly into tables.
- It can scale read workloads by adding replica instances.
- Managed backups and failover reduce the operational work required to run a database.
- Existing MongoDB-based applications may be able to use DocumentDB with limited code changes, but compatibility should be checked before migration.

> **Analogy:** A relational database is like a spreadsheet with strict columns; DocumentDB is like a set of smart folders where each file can contain different details.

> **Important:** DocumentDB runs inside a VPC and is normally not exposed directly to the public internet. Applications usually connect through private networking and security-group rules.


## Steps to create documentDB cluster

1. Open the AWS Console, search for **Amazon DocumentDB**, and select **Clusters**.
2. Select **Create** and choose the creation method:
	- **Standard create:** Shows all configuration options and is the best choice for learning or production planning.
	- **Easy create:** Uses recommended defaults and is useful for a quick trial.
3. Under **Cluster configuration**, choose the engine version and enter a **Cluster identifier**, such as `my-documentdb-cluster`.
4. Configure the administrator credentials:
	- Enter a **Master username** for the database administrator.
	- Choose whether DocumentDB should create the password or enter your own secure password.
	- Store the credentials in AWS Secrets Manager or another approved secret store instead of placing them in application code.
5. Choose the instance settings:
	- Select an **Instance class** based on memory and workload requirements.
	- Set the number of instances. One instance is suitable for testing; multiple instances improve availability and provide read capacity.
	- Choose a **Primary** instance for writes and optional **Replica** instances for reads and failover.
6. Configure **Connectivity**:
	- Select the correct **VPC** where the application runs.
	- Choose a **DB subnet group** covering private subnets in multiple Availability Zones.
	- Select a **VPC security group** that allows MongoDB-compatible traffic on port `27017` only from approved application resources.
	- Keep **Public accessibility** disabled unless there is a specific, carefully reviewed requirement.
7. Under **Additional configuration**, review the following options:
	- **Port:** Keep the default `27017` unless the application requires another port.
	- **Encryption:** Enable encryption at rest and use the AWS-managed KMS key for a simple setup, or a customer-managed key for additional control.
	- **Backup retention:** Choose how many days automated backups should be kept.
	- **Preferred backup and maintenance windows:** Choose periods when backups or maintenance are least disruptive.
	- **Deletion protection:** Enable it for important environments to prevent accidental deletion.
	- **Log exports:** Enable useful audit or profiler logs and send them to CloudWatch Logs when troubleshooting or monitoring is required.
8. Review the estimated cost and all networking, credential, backup, and encryption settings.
9. Select **Create cluster**.
10. Wait until the cluster and its primary instance show **Available**, then copy the cluster endpoint.
11. Configure the application security group and connect using a MongoDB-compatible driver over TLS.

> **Why these settings matter:** The cluster settings define the database, instance settings control performance and availability, and VPC settings control who can connect. Backups, encryption, deletion protection, and logs protect the data and make the cluster easier to operate.

> **Beginner-friendly setup:** Use **Standard create**, a small instance, one primary instance for testing, private subnets, public access disabled, port `27017`, encryption enabled, and a security group that allows access only from your application.

### Connect App Runner to DocumentDB

> **Why is this needed?** DocumentDB is private inside a VPC, while App Runner runs outside that VPC by default. Without a VPC connector, the App Runner service cannot reach the DocumentDB endpoint. The connector creates a private outbound path into the VPC, and security groups limit that path to approved traffic on port `27017`.

1. **Confirm the network:** Make sure the App Runner VPC connector and the DocumentDB cluster are in the same VPC, or that the VPCs are connected through approved networking such as peering or Transit Gateway.
2. **Create a connector security group:** Create a security group for the App Runner VPC connector. Allow outbound traffic required by the application.
3. **Update the DocumentDB security group:** Add an inbound rule for TCP port `27017` with the App Runner connector security group as the source. Do not allow `0.0.0.0/0`.
4. **Create the App Runner VPC connector:**
	- Open **App Runner** in the AWS Console.
	- Open **VPC connectors** and select **Create VPC connector**.
	- Enter a connector name, such as `documentdb-connector`.
	- Select the same VPC used by DocumentDB.
	- Select private subnets in at least two Availability Zones.
	- Select the connector security group.
	- Choose **Create** and wait for the connector to become available.
5. **Attach the connector to App Runner:** Open the App Runner service, choose **Configuration** or **Networking**, select the VPC connector for outgoing traffic, and save or deploy the change.
6. **Configure application settings:** Add the DocumentDB cluster endpoint, port `27017`, database name, username, and password as environment variables or secrets. Do not hard-code credentials in the image or source code.
7. **Enable TLS:** Include the DocumentDB CA certificate in the application image or retrieve it securely, then configure the MongoDB-compatible client to use TLS when connecting.
8. **Use the cluster endpoint:** Connect to the DocumentDB cluster endpoint rather than an instance-specific endpoint. Include any DocumentDB driver options required by the application, such as `replicaSet=rs0` and `retryWrites=false`.
9. **Deploy and test:** Deploy the App Runner service, check application and App Runner logs, and verify that the application can connect to DocumentDB and perform a simple read or write.


## Tips for debugging AppRunner

- **Check the deployment status:** App Runner may roll back configuration changes when health checks fail. Confirm that the updated VPC connector, environment variables, port, and start command are still present after deployment.
- **Verify the network:** Confirm that the App Runner VPC connector and DocumentDB cluster use the intended VPC and that the connector subnets can route to the DocumentDB subnets.
- **Check security groups:** Confirm that the DocumentDB security group allows inbound TCP traffic on port `27017` from the App Runner connector security group. Avoid allowing `0.0.0.0/0`.
- **Review application logs:** Open the App Runner logs, select the time range that includes the failed deployment or request, and look for connection timeouts, DNS errors, TLS errors, authentication failures, or application crashes.
- **Verify connection settings:** Check that the application uses the DocumentDB cluster endpoint, the correct port, valid credentials, and the required TLS and driver options.
- **Test after each change:** Make one configuration change at a time, redeploy, and confirm both the App Runner health check and a simple DocumentDB read or write.