# About Greenplum for Kubernetes

{{#include ./product_name.md}} utilizes the Kubernetes platform to help you quickly and reliably deploy Greenplum clusters that are tailored for a variety of use cases, such as analytical processing with business intelligence tools or high-performance ETL operations for data scientists. Post-deployment, {{#include ./product_name.md}} helps you maintain your Greenplum cluster by expanding it to accommodate additional processing requirements or activating standby servers to maintain high availability.  

The {{#include ./product_name.md}} distribution for Kubernetes provides Docker images with the {{#include ./product_name.md}} Database software and the Greenplum Operator running on Ubuntu. The Greenplum Operator registers {{#include ./product_name.md}} as a first-class resource within Kubernetes, and orchestrates the deployment, management, and deletion of Greenplum in Kubernetes, as directed by your requests. In most cases, you simply specify the desired Greenplum configuration in a declarative YAML-formatted file, and provide that file to the Greenplum Operator.

## <a id="differences"></a>Differences from Greenplum Deployed on Hardware or VMs

{{#include ./product_name_long.md}} is functionally different from the {{#include ./product_name.md}} Database in other deployment environments:

- Certain features, such as MADlib, that are optional in other deployments are installed automatically with {{#include ./product_name_long.md}}.
- {{#include ./product_name_long.md}} uses Resource Group-based resource management by default, compared to other deployment environments that use Resource Queue-based resource management.
- {{#include ./product_name_long.md}} sets the default value of the `gp_resource_group_memory_limit` configuration parameter to 1.0, because it operates in an environment with reserved resources that are not shared between multiple primary and mirror segments.
- {{#include ./product_name_long.md}} does not currently support cluster monitoring with Greenplum Command Center. Use system-level monitoring tools such as Prometheus and Grafana until Greenplum Command Center support is available.
- {{#include ./product_name.md}} clusters in Kubernetes do not support installing Greenplum extensions that use the `.gppkg` format (and `gppkg` utility). Future releases will include these extensions as part of the distribution, as with MADlib.