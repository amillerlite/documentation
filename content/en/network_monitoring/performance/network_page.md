---
title: Network Page
kind: documentation
description: Explore your Network data between each source and destination across your stack.
aliases:
    - /network_performance_monitoring/network_table
    - /network_performance_monitoring/network_page
further_reading:
    - link: 'https://www.datadoghq.com/blog/network-performance-monitoring'
      tag: 'Blog'
      text: 'Network Performance Monitoring'
    - link: '/network_monitoring/devices'
      tag: 'Documentation'
      text: 'Network Device Monitoring'
    - link: '/network_monitoring/performance/setup'
      tag: 'Documentation'
      text: 'Collect your Network Data with the Datadog Agent.'
---

{{< img src="network_performance_monitoring/network_page/main_page_npm.png" alt="Main page" >}}

## Queries

To refine your search to traffic between particular endpoints, aggregate and filter your network aggregate connections **with tags**. You can select tags for the **_source_** and **_destination_** by using the search bar at the top of the page.

The following screenshot shows the default view, which aggregates the _source_ and _destination_ by the `service` tag. Accordingly, each row in the table represents service-to-service aggregate connections when aggregated over a one hour time period.

{{< img src="network_performance_monitoring/network_page/context_npm.png" alt="context" style="width:80%;">}}

The next example shows all aggregate connections from IP addresses representing services in region `us-east-1` to availability zones:

{{< img src="network_performance_monitoring/network_page/flow_table_region_az.png" alt="Aggregate connection table filtered" style="width:80%;">}}

You can set the timeframe over which traffic is aggregated using the time selector at the top right of the page:

{{< img src="network_performance_monitoring/network_page/npm_timeframe.png" alt="Time frame NPM" style="width:30%;">}}

Tags from Datadog integrations or Unified Service Tagging can be used for aggregating and filtering automatically. See [custom facets](#custom-facets), below, for other tags. You can also select "Popular Tags" to aggregate on, using a combination of `service`, `kube_service`, `short_image`, `container_name`, and other common tags.

### Facet panels

Facet panels mirror the tags in your search bar query. Switch between the facet panels with the _Source_ and _Destination_ tabs on top:

{{< img src="network_performance_monitoring/network_page/destination_panel.png" alt="Destination panel" style="width:20%;">}}

#### Custom facets

Aggregate and filter your traffic data by any tags in Datadog network page. An include list of tags is provided by default, which you can find in the search bar dropdown menu:

{{< img src="network_performance_monitoring/network_page/drop_down_npm.png" alt="Dropdown menu" style="width:90%;">}}

Include listed tags are `service`, `availability zone`, `env`, `environment`, `pod`, `host`, `ip`, and `port`, among others. If you want to aggregate or filter traffic by a tag that is not already in the menu, add it as a custom Facet:

1. Select the `+` button on the top right of the facet panels.
2. Enter the relevant tag you want to create a custom facet upon.
3. Click `Create`.

Once the custom facet is created, use this tag to filter and aggregate traffic in the network page and map. All custom facets can be viewed in the bottom `Custom` section of the facet panels.

### Wildcard search
To perform a multi-character wildcard search, use the `*` symbol as follows:

- `service:web*` matches all services that start with web
- `service:*web` matches all services that end with web
- `service:*web*` matches all services that contain the string web

Wildcard searches work within facets with this syntax. This query returns all the services that end with the string mongo:

`service:*mongo`

To learn more, see the [search syntax][1] documentation.

## Network data

{{< img src="network_performance_monitoring/network_page/network_data.png" alt="network data" style="width:90%;" >}}

Your network metrics are displayed through the graphs and the associated table. All sent and received metrics are displayed from the perspective of the source:

* **Sent metrics**: measure the value of something from the _source_ to the _destination_ from the source's perspective.
* **Received metrics**: measure the value of something from the _destination_ to the _source_ from the source's perspective.

Values displayed might be different for `sent_metric(source to destination)` and `received_metric(destination to source)` if there is a large number of packet drops. In this case, if the `destination` sends a lot of bytes to the `source`, the aggregate connections that originate at `destination` include those bytes, but the aggregate connections that originate at `source` do not see them as received.

**Note:** Data is collected every 30 seconds, aggregated in five minute buckets, and retained for 14 days.

### Metrics

#### Network load

The following network load metrics are available:

| Metric          |  Description                                                                                                                                    |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **Volume**      | The number of bytes sent or received over a period. Measured in bytes (or orders of magnitude thereof) bidirectional.                           |
|  **Throughput** | The rate of bytes sent or received over a period. Measured in bytes per second, bidirectional.                                                  |

#### TCP

TCP is a connection-oriented protocol that guarantees in-order delivery of packets. The following TCP metrics are available:

| Metric                    |  Description                                                                                                                           |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| **TCP Retransmits** | TCP Retransmits represent detected failures that are retransmitted to ensure delivery. Measured in count of retransmits from the `source`. |
| **TCP Latency** | Measured as TCP smoothed round-trip time, that is the time between a TCP frame being sent and acknowledged. |
| **TCP Jitter** | Measured as TCP smoothed round-trip time variance. |
| **Established Connections** | The number of TCP connections in an established state. Measured in connections per second from the `source`. |
| **Closed Connections** | The number of TCP connections in a closed state. Measured in connections per second from the `source`. |

### Cloud service autodetection

If you're relying on managed cloud services like S3 or Kinesis, you can monitor the performance of traffic to those services from your internal applications. Scope your view to a particular AWS or Google Cloud dependency to pinpoint latency, assess database performance, and visualize your network more completely.

{{< img src="network_performance_monitoring/network_page/cloud-service-hero-docs.png" alt="Cloud Service Map" >}}

For instance, you can

- visualize data flow from your internal Kubernetes cluster to `service:aws.s3` in the [Network Map][2]
- pivot to the [Network Page](#table) to isolate which pods are establishing the most connections to that service, and
- validate that their request are successful by analyzing S3 performance metrics, which are correlated with traffic perfromance directly in the sidepanel for a given dependency, under the *Integration Metrics* tab.

NPM automatically maps

- network calls to S3 (which can broken down by `s3_bucket`), RDS (which can be broken down by `rds_instance_type`), Kinesis, ELB, Elasticache, and other [AWS services][3].
- API calls to AppEngine, Google DNS, Gmail, and other [Google Cloud services][4].

To monitor other endpoints where an Agent cannot be installed (such as public APIs), group the destination in the Network Overview by the [`domain` tag](#domain-resolution). Or, see the section below for cloud service resolution.

### Cloud service enhanced resolution
If you have [setup][9] enhanced resolution for AWS or Azure, NPM can filter and group network traffic with several resources collected from these cloud providers. Depending on the cloud provider and resource, you have different sets of tags available to query with. Azure loadbalancers only have user-defined tags. For AWS, Datadog applies the tags defined below in addition to the user-defined tags.

 #### Amazon Web Services
 {{< tabs >}}
 {{% tab "Loadbalancers" %}}
 - name
 - loadbalancer
 - load_balancer_arn
 - dns_name (format loadbalancer/dns:)
 - region
 - account_id
 - scheme
 - custom (user-defined) tags applied to AWS Loadbalancers
 {{% /tab %}}

 {{% tab "NAT Gateways" %}}
 - gateway_id
 - gateway_type
 - aws_nat_gateway_id
 - aws_nat_gateway_public_ip
 - aws_account
 - availability-zone
 - region
 - custom (user) tags applied to AWS Nat Gateways
 {{% /tab %}}

 {{% tab "VPC Internet Gateway" %}}
 - gateway_id
 - gateway_type
 - aws_internet_gateway_id
 - aws_account
 - region
 - custom (user) tags applied to VPC Internet Gateways
 {{% /tab %}}

{{% tab "VPC Endpoint" %}}
 - gateway_id
 - gateway_type
 - aws_vpc_endpoint_id
 - custom (user) tags applied to VPC Internet Endpoints
 {{% /tab %}}

 {{< /tabs >}}

### Domain resolution

Starting with Agent 7.17+, the Agent resolves IPs to human-readable domain names for external and internal traffic. Domain allows you to monitor cloud provider endpoints where a Datadog Agent cannot be installed, such as S3 buckets, application load balancers, and APIs. Unrecognizable domain names such as DGA domains from C&C servers may point to network security threats. `domain` **is encoded as a tag in Datadog**, so you can use it in search bar queries and the facet panel to aggregate and filter traffic.

{{< img src="network_performance_monitoring/network_page/domain_aggregation.png" alt="Domain aggregation" >}}

**Note**: DNS resolution is supported for hosts where the system probe is running on the root network namespace, which is usually caused by running the system-probe in a container without using the host network.

### Network Address Translation (NAT)

NAT is a tool used by Kubernetes and other systems to route traffic between containers. When investigating a specific dependency (for example, service to service), you can use the presence or absence of pre-NAT IPs to distinguish between Kubernetes-native services, which do their own routing, and services that rely on external clients for routing. This feature does not currently include resolution of NAT gateways.

To view pre-NAT and post-NAT IPs, use the _Show pre-NAT IPs_ toggle in the table settings. When this setting is toggled off, IPs shown in the Source IP and Dest IP columns are by default post-NAT IPs. In cases where you have multiple pre-NAT IPs for one post-NAT IP, the top 5 most common pre-NAT IPs will be displayed. `pre_nat.ip` is a tag like any other in the product, so you can use it to aggregate and filter traffic.

{{< img src="network_performance_monitoring/network_page/prenat_ip.png" alt="pre-NAT IPs" >}}

### Network ID

NPM users may configure their networks to have overlapping IP spaces. For instance, you may want to deploy in multiple VPCs (virtual private clouds) which have overlapping address ranges and communicate only through load balancers or cloud gateways.

To correctly classify traffic destinations, NPM uses the concept of a network ID, which is represented as a tag. A network ID is an alphanumeric identifier for a set of IP addresses that can communicate with one another. When an IP address mapping to several hosts with different network IDs is detected, this identifier is used to determine the particular host network traffic is going to or coming from.

In AWS and Google Cloud, the network ID is automatically set to the VPC ID. For other environments, the network ID may be set manually, either in `datadog.yaml` as shown below, or by adding the `DD_NETWORK_ID` to the process and core Agent containers.

  ```yaml
  network:
     Id: <your-network-id>
  ```

### Saved views

Organize and share views of traffic data. Saved Views make debugging faster and empower collaboration. For instance, you can create a view, save it for the future for common queries, and copy its link to share network data with your teammates.

{{< img src="network_performance_monitoring/network_page/npm_saved_views.png" alt="Saved Views" >}}

- To save a view: click the *+ Save* button and name the view to record your current query, table configuration, and graph metric selections.
- To load a view: click *Views* at the top left to see your Saved Views and select a view from the list.
- To rename a view: hover over a view in the Saved Views list and click the gear icon to *Edit name*.
- To share a view: hover over a view in the Saved Views list and click the link icon to *Copy permalink*.

To learn more, see the [Saved Views][5] documentation.


## Table

The network table breaks down the _Volume_, _Throughput_, _TCP Retransmits_, _Round-trip Time (RTT)_, and _RTT variance_ metrics between each _source_ and _destination_ defined by your query.

{{< img src="network_performance_monitoring/network_page/network_table.png" alt="Data table" >}}

You can configure the columns in your table using the `Customize` button at the top right of the table.

Configure the traffic shown with the `Filter Traffic` button.

{{< img src="network_performance_monitoring/network_page/filter_traffic_toggles_v2.png" alt="Flow Details" style="width:80%;">}}

External traffic (to public IPs) and Datadog Agent traffic is shown by default. To narrow down your view, you can choose to toggle off the `Show Datadog Traffic` and `Show External Traffic` toggles.

### Unresolved traffic

Unresolved source and destination tags are marked as `N/A`. A traffic source or destination endpoint may be unresolved because:

* The host or container source or destination IPs are not tagged with the source or destination tags used for traffic aggregation.
* The endpoint is outside of your private network, and accordingly is not tagged by the Datadog Agent.
* The endpoint is a firewall, service mesh or other entity where a Datadog Agent cannot be installed.

Use the _Show N/A (Unresolved Traffic)_ toggle in the upper right corner of the data table to filter out aggregate connections with unresolved (`N/A`) sources or destinations.

Select any row from the data table to see associated logs, traces, and processes for a given _source_ <=> _destination_ aggregate connection:

{{< img src="network_performance_monitoring/network_page/flow_details.png" alt="Aggregate Connection Details" style="width:80%;">}}

## Sidepanel

The sidepanel provides contextual telemetry to help you debug network dependencies. Use the Flows, Logs, Traces, and Processes tabs to determine whether a high retransmit count or latency in traffic between two endpoints is due to:
- A spike in traffic volume from a particular port or IP.
- Heavy processes consuming the CPU or memory of the destination endpoint.
- Application errors in the code of the source endpoint.

{{< img src="network_performance_monitoring/network_page/npm_sidepanel.png" alt="Flow Details" style="width:80%;">}}

### Common tags

The top of the sidepanel displays common source and destination tags shared by the inspected dependency's most recent connections. Use common tags to gain additional context into a faulty endpoint. For instance, when troubleshooting latent communication to a particular service, common destination tags will surface:
- Granular context such as the container, task, or host to which traffic is flowing.
- Wider context such as the availability zone, cloud provider account, or deployment in which the service runs.

### Security

The **Security** tab highlights potential network threats and findings detected by [Cloud Workload Security][6] and [Cloud Security Posture Management][7]. These signals are generated when Datadog detects network activity that matches a [detection or compliance rule][8], or if there are other threats and misconfigurations related to the selected network flow.

<div class="alert alert-warning">Network threat detections is in private beta. Fill out this <a href="https://forms.gle/zjfbxB7Cqjxj5R5h7">form</a> to request access.</div>

## Further Reading

{{< partial name="whats-next/whats-next.html" >}}

[1]: /logs/search_syntax/
[2]: /network_monitoring/performance/network_map/
[3]: /network_monitoring/performance/guide/aws_supported_services/
[4]: /network_monitoring/performance/guide/gcp_supported_services/
[5]: /logs/explorer/saved_views/
[6]: /security/cloud_workload_security/
[7]: /security/cspm/
[8]: /security/detection_rules/
[9]: /network_monitoring/performance/setup/#enhanced-resolution
