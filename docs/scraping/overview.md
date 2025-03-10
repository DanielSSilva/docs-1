# Metrics Declaration

All the Azure Monitor metrics that need to be scraped are consolidated in one YAML
file which is referred to as the metric declaration.

This declaration defines the overall Azure metadata and all the metrics you want to expose.

Every metric describes the Azure Monitor metric that it represents and what Azure resources that should be scraped.
 It allows you to statically declaring the resources to scrape and/or use [automatic resource discovery](https://promitor.io/concepts/#using-resource-discovery).

## Supported Azure Services

[Generic Azure Resource](providers/generic-azure-resource) allows you to scrape every Azure
service supported by Azure Monitor.

## Supported Azure Services

[Generic Azure Resource](providers/generic-azure-resource) allows you to scrape every Azure
service supported by Azure Monitor.

We also provide a simplified way to scrape the following Azure resources:

- [Azure API Management](api-management)
- [Azure Application Gateway](application-gateway)
- [Azure Application Insights](application-insights)
- [Azure App Plan](app-plan)
- [Azure Cache for Redis](redis-cache)
- [Azure Cache for Redis Enterprise](redis-enterprise-cache)
- [Azure Content Delivery Network (CDN)](cdn)
- [Azure Container Instances](container-instances)
- [Azure Container Registry](container-registry)
- [Azure Cosmos DB](cosmos-db)
- [Azure Data Factory](data-factory)
- [Azure Data Share](data-share)
- [Azure Database for PostgreSQL](postgresql)
- [Azure Database for MariaDB](maria-db)
- [Azure Database for MySQL](mysql)
- [Azure Event Hubs](event-hubs)
- [Azure Express Route Circuit](express-route-circuit)
- [Azure Front Door](front-door)
- [Azure Function App](function-app)
- [Azure IoT Hub](iot-hub)
- [Azure IoT Hub Device Provisioning Service (DPS)](iot-hub-device-provisioning-service)
- [Azure Key Vault](key-vault)
- [Azure Kubernetes Service](kubernetes)
- [Azure Load Balancer](load-balancer)
- [Azure Logic Apps](logic-apps)
- [Azure Monitor Autoscale](monitor-autoscale)
- [Azure Network Gateway](network-gateway)
- [Azure Network Interface](network-interface)
- [Azure Service Bus Namespace](service-bus-namespace)
- [Azure SQL Database](sql-database)
- [Azure SQL Elastic Pool](sql-elastic-pool)
- [Azure SQL Managed Instance](sql-managed-instance)
- [Azure SQL Server](sql-server)
- [Azure Storage (Account)](storage-account)
- [Azure Storage (Blob)](blob-storage)
- [Azure Storage (Files)](file-storage)
- [Azure Storage (Queue)](storage-queue)
- [Azure Synapse (Apache Spark pool)](synapse-apache-spark-pool)
- [Azure Synapse (SQL pool)](synapse-sql-pool)
- [Azure Synapse (Workspace)](synapse-workspace)
- [Azure Virtual Machine](virtual-machine)
- [Azure Virtual Machine Scale Set (VMSS)](virtual-machine-scale-set)
- [Azure Virtual Network](virtual-network)
- [Azure Web App](web-app)

Want to help out? Create an issue and [contribute a new scraper](https://github.com/tomkerkhove/promitor/blob/master/adding-a-new-scraper.md).

## General Declaration

As Promitor evolves we need to change the structure of our metrics declaration.

`version: {version}` - Version of declaration that is used. Allowed
values are `v1`. *(Required)*

### Azure

- `azureMetadata.tenantId` - The id of the Azure tenant that will be queried.
- `azureMetadata.subscriptionId` - The id of the default subscription to query.
- `azureMetadata.resourceGroupName` - The name of the default resource group to query.
- `azureMetadata.cloud` - The name of the Azure cloud to use. Options are `Global`
 (default), `China`, `UsGov` & `Germany`.

### Metric Defaults

- `metricDefaults.aggregation.interval` - The default interval which defines over
  what period measurements of a metric should be aggregated.
  a cron that fits your needs.
- `metricDefaults.limit` - The default maximum amount of resources to scrape when using dimensions
  or filters.
- `metricDefaults.labels` - The default labels that will be applied to all metrics. _(starting as of v2.3)_
- `metricDefaults.scraping.schedule` - A cron expression that controls
  the frequency of which all the configured metrics will be scraped from Azure Monitor.
  You can use [crontab-generator.org](https://crontab-generator.org/) to generate
  a cron that fits your needs. *(Required)*

### Metrics

Every metric that is being declared needs to define the following fields:

- `name` - Name of the metric that will be reported.
- `description` - Description for the metric that will be reported.
- `resourceType` - Defines what type of resource needs to be queried.
- `azureMetricConfiguration.metricName` - The name of the metric in Azure Monitor
  to query
- `azureMetricConfiguration.aggregation.type` - The aggregation that needs to be
  used when querying Azure Monitor
- `azureMetricConfiguration.aggregation.interval` - Overrides the default aggregation
  interval defined in `metricDefaults.aggregation.interval` with a new interval
- `resources` - An array of one or more resources to get metrics for. The fields
  required vary depending on the `resourceType` being created, and are documented
  for each resource.
- `azureMetricConfiguration.limit` - The maximum amount of resources to scrape when using dimensions
  or filters.
- `resourceDiscoveryGroups` An array of one or more resource discovery groups that will be used to automatically
 discover all resources through Promitor Resource Discovery. For every found resource, it will get the metrics and
  report them. Learn more on resource discovery, in [our documentation](https://promitor.io/concepts#using-resource-discovery)

All resources provide the capability to override the default Azure metadata:

- `subscriptionId` - Changes the subscription id to which the resource belongs. _(Overrides `azureMetadata.subscriptionId`)_
- `resourceGroupName` - Changes the resource group that contains resource. (Overrides `azureMetadata.resourceGroupName`)

Additionally, the following fields are optional:

- `azureMetricConfiguration.dimension.name` - The name of the dimension that should
   be used to scrape a multi-dimensional metric in Azure Monitor.
  - ☝ *Promitor simply acts as a proxy and will not validate if it's supported or
     not, we recommend verifying that the dimension is supported in the
     [official documentation](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/metrics-supported)*
- `labels` - Defines a set of custom labels to include for a given metric.
- `scraping.schedule` - A scraping schedule for the individual metric; overrides
  the the one specified in `metricDefaults`

### Example

Here is an example of how you can scrape two Azure Service Bus queues  in different
resource groups, one in the `promitor` resource group and one on the `promitor-dev`
resource group:

```yaml
version: v1
azureMetadata:
  tenantId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
  subscriptionId: yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy
  resourceGroupName: promitor
  cloud: China
metricDefaults:
  aggregation:
    interval: 00:05:00
  limit: 10
  labels:
    geo: china
    environment: dev
  scraping:
    # Every minute
    schedule: "0 * * ? * *"
metrics:
  - name: azure_service_bus_active_messages
    description: "The number of active messages on a service bus queue."
    resourceType: ServiceBusNamespace
    labels:
      app: promitor
      tier: messaging
    scraping:
      # Every 2 minutes
      schedule: "0 */2 * ? * *"
    azureMetricConfiguration:
      metricName: ActiveMessages
      limit: 5
      dimension:
        name: <dimension-name>
      aggregation:
        type: Total
        interval: 00:15:00
    resources: # Optional, required when no resource discovery is configured
      - namespace: promitor-messaging
        queueName: orders
      - namespace: promitor-messaging-dev
        resourceGroupName: promitor-dev
        subscriptionId: ABC
    resourceDiscoveryGroups: # Optional, requires Promitor Resource Discovery agent (https://promitor.io/concepts/how-it-works#using-resource-discovery)
    - name: service-bus-landscape
```

[&larr; back](/)
