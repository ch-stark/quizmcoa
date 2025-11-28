Here is the raw Markdown code for the Q&A review. You can copy the content inside the code block below and save it as a .md file.

Markdown

# MCOA Metrics Documentation Review

## Part 1: Review Questions (25)

This section covers architecture, configuration, troubleshooting, and best practices.

### General Architecture & Enabling

**1. What are the three key advantages of the MCOA metrics collection compared to the previous version?**
> Improved Configurability, Increased Metrics Federation Scalability, and More Performant Remote Write with Network Partition Resiliency.

**2. Which two upstream components replace the legacy Endpoint operator and Metrics Collector?**
> The COO's Prometheus Operator and a Prometheus Agent.

**3. What is the required configuration change in the MultiClusterObservability custom resource to enable default platform metrics?**
> Set `spec.capabilities.platform.metrics.default.enabled` to `true`.

**4. In which namespace is the multicluster-observability-addon-manager deployed?**
> `open-cluster-management-observability`.

**5. What is the default mechanism MCOA uses to ensure consistency and enforce invariants in configurations?**
> Server-side apply.

### Configuration APIs

**6. What are the three main APIs MCOA offers for configuring metrics collection?**
> `PrometheusAgents`, `ScrapeConfigs`, and `PrometheusRules`.

**7. If a new placement is added to the ClusterManagementAddon, what does the addon manager automatically create?**
> A specific default `PrometheusAgent`, default `ScrapeConfigs`, and `PrometheusRules` for that placement.

**8. What happens if a user modifies an enforced field in a PrometheusAgent, such as removing the remote write configuration?**
> The addon manager automatically reverts the change.

**9. Which API resource is used to define what metrics to collect?**
> `ScrapeConfig`.

**10. What labels must be present on a PrometheusAgent for the addon manager to recognize it?**
> * `app.kubernetes.io/component`
> * `app.kubernetes.io/managed-by`
> * `placement-ref-name`
> * `placement-ref-namespace`

### Customizing Metrics & Workloads

**11. What is the default scrape interval set in the PrometheusAgent?**
> 300 seconds.

**12. What error might occur if the scrape interval is set below 90 seconds?**
> "Error on ingesting out-of-order samples."

**13. Which label key distinguishes between a platform metrics collector and a user-workload metrics collector?**
> `app.kubernetes.io/component`.

**14. How can you restrict user workload ScrapeConfigs to only collect metrics from specific namespaces (e.g., those with `app: my-app`)?**
> By configuring the `scrapeConfigNamespaceSelector` in the `PrometheusAgent`.

**15. What specific annotation is required for PrometheusRules targeting user workloads to ensure they are deployed to the correct namespace?**
> `observability.open-cluster-management.io/target-namespace`.

**16. Why must you use the monitoring.coreos.com group for PrometheusRule resources, while ScrapeConfig uses monitoring.rhobs?**
> The document explicitly warns to use `monitoring.coreos.com` for `PrometheusRules` and `monitoring.rhobs` for `ScrapeConfig` and `PrometheusAgent`.

### Advanced Features & External Export

**17. What are the two advantages of configuring a remote write to an external endpoint directly on the PrometheusAgent rather than using writeStorage on the Hub?**
> Configurable metrics forwarding (using relabeling) and enhanced resiliency (up to two hours).

**18. How can you filter out specific metrics (like the Watchdog alert) before they are sent to the hub?**
> By adding `writeRelabelConfigs` with a `drop` action in the `remoteWrite` configuration of the `PrometheusAgent`.

**19. When exporting metrics to an external endpoint via PrometheusAgent, where must the TLS secrets be created?**
> In the `open-cluster-management-observability` namespace.

### Troubleshooting & Operations

**20. What happens if a managed cluster belongs to multiple placements referenced in the ClusterManagementAddon?**
> It inherits the configuration from the **last compatible placement** listed in the `ClusterManagementAddon`.

**21. How long does the standard remote write implementation provide resiliency to network partitions between spokes and the hub?**
> Up to one hour.

**22. What are the three specific alerting rules pushed to managed clusters to monitor the addon's health?**
> 1. `MetricsCollectorNotIngestingSamples`
> 2. `MetricsCollectorRemoteWriteFailures`
> 3. `MetricsCollectorRemoteWriteBehind`

**23. How can you disable metrics collection on a specific cluster?**
> By excluding the cluster from all placements referenced in the `ClusterManagementAddOn` (e.g., using a label selector/predicate).

**24. When troubleshooting a custom ScrapeConfig, which field in the ClusterManagementAddOn status confirms that the config reference exists?**
> The `specHash` within `status.installProgressions.configReferences`.

**25. What additional resources does MCOA deploy on non-OpenShift managed clusters that are not deployed on OpenShift clusters?**
> The Node Exporter, Kube State Metrics, and a Prometheus server.

---

## Part 2: MCOA Deep Dive Quiz (50 Questions)

### Core Identity, Purpose, and Status

**1. What is the primary function of the MultiCluster Observability (MCO) component within RHACM?**
> MCO allows users to gain insights and optimize RHACM managed clusters.

**2. What is the MultiCluster Observability Addon (MCOA) described as?**
> MCOA is an evolution of the MultiCluster Observability Operator, focusing on providing a more efficient metrics collection architecture for multicluster environments.

**3. What are the three primary observability signals (data types) that MCOA is designed to orchestrate and include collection, storage, authorization (authz), and UI for?**
> Metrics, Logs, and Traces.

**4. In which major RHACM version was the Multi-cluster Observability Addon (MCOA) for Metrics listed as being generally available (GA)?**
> ACM 2.15.

**5. In which RHACM version was the MCOA component initially delivered, hidden behind a feature flag?**
> ACM 2.12.

**6. MCOA re-uses the APIs created for what type of user stories?**
> Single OpenShift Container Platform (OCP) user stories.

**7. When enabled for metrics collection, what older components does MCOA replace on the managed cluster side?**
> The Endpoint operator and the Metrics Collector.

**8. What key element of the existing observability stack remains unchanged when MCOA is enabled for metrics collection?**
> Thanos storage.

### Benefits and Improvements

**9. What is the main benefit MCOA provides regarding configurability compared to the previous metrics stack?**
> Improved configurability by enabling users to directly configure custom resources deployed on spokes using standard upstream Prometheus operator APIs.

**10. How does MCOA achieve increased metrics federation scalability?**
> Through sharded metrics federation using distinct `ScrapeConfigs`, each independently federated from the in-cluster Prometheus.

**11. What industry-standard implementation does MCOA use to send metrics efficiently from spokes to the hub?**
> Standard `remoteWrite` implementation.

**12. MCOA's architecture provides resiliency to network partitions between the spoke and hub for how long without data loss?**
> Up to 2 hours.

**13. What improvement related to metrics collection ensures there are "No More Gaps" in data, providing continuous visibility?**
> Full-Time Pod Granularity, ensuring all dashboards continuously display metrics at the individual pod level (24/7), resolving previous issues where critical pod-level metrics were only available during alerts.

**14. What are MCOA's default platform metrics optimized and curated for?**
> Performance, while maintaining pod-level observability and minimizing cardinality.

### Architecture and Components

**15. What is the new metrics collector workload running on managed clusters (spokes) that replaces the legacy Metrics Collector?**
> The Prometheus Agent.

**16. What configuration API resources are used in MCOA to define metrics collection configuration, replacing the legacy Allow list ConfigMap?**
> `ScrapeConfig` CRs and `PrometheusRule` CRs.

**17. On the hub cluster, what new component is deployed when enabling MCOA?**
> The `multicluster-observability-addon-manager`.

**18. What API is used by MCOA to define precisely what configuration is dedicated to what cluster, acting as a multicluster scheduler?**
> The Placement API from Open Cluster Management (OCM).

**19. Which configuration API in MCOA specifies options like the installation namespace or node selector for the deployed addon?**
> `AddonDeploymentConfig`.

**20. What is the default installation namespace for MCOA resources deployed on managed clusters (spokes)?**
> `open-cluster-management-agent-addon`.

**21. For non-OCP managed clusters, MCOA supports the deployment of which three additional components compared to OCP clusters?**
> The Node Exporter, Kube State Metrics, and a Prometheus server.

### Configuration and Management

**22. When enabling MCOA in the MultiClusterObservability CR, which specific metric type is noted as being required to be enabled?**
> Platform metrics (e.g., `spec.capabilities.platform.metrics.default.enabled: true`).

**23. When adding custom metrics via the MCOA configuration, what specific resource type replaces the use of the legacy `metrics_list.yaml`?**
> `ScrapeConfig`.

**24. When adding custom recording rules within MCOA, which Kubernetes Custom Resource Definition (CRD) should be used?**
> `PrometheusRule` (specifically from the `monitoring.coreos.com` group).

**25. What operation does MCOA use to track and automatically revert modifications if a user attempts to change an enforced configuration field in a Prometheus Agent?**
> Server-side apply.

**26. In the structure used for migrating custom metric allow lists (Names, Matches, Recording_rules, Collect_rules, Renames), which two capabilities are explicitly stated as "not supported" in MCOA?**
> `Renames` and `Collect_rules`.

**27. If a ScrapeConfig for user workload metrics is deployed in a specific namespace on the spoke cluster, what restriction is enforced on the metrics collected?**
> It will only collect metrics coming from that specific namespace, enforcing the namespace label on the collected metrics.

**28. To limit metrics collection to spokes running Single Node OpenShift (SNO), what mechanism, using placement predicates, can be used to identify these clusters?**
> Selecting `ManagedClusters` with the label `vendor: OpenShift` and the claim with name `controlplanetopology.openshift.io` and value `SingleReplica`.

**29. If a managed cluster is associated with multiple placements referenced in the ClusterManagementAddon, whose configuration will the cluster inherit?**
> The configuration from the **last compatible placement** listed in the `ClusterManagementAddon`.

**30. To entirely disable metrics collection on a specific managed cluster, what configuration action is required regarding placements?**
> The cluster must be excluded from all placements referenced in the `ClusterManagementAddOn`.

### Alerting and Logs

**31. By default, what is the policy regarding alert forwarding from OCP managed clusters to the Hub Alertmanager when using MCOA?**
> By default, MCOA does not configure alert forwarding automatically.

**32. What method is recommended for enabling alert forwarding in MCOA, overcoming the default disabled behavior?**
> Using an RHACM Policy (specifically a `ConfigurationPolicy` applied via Policy).

**33. MCOA supports Traces collection and forwarding based on which technology/build?**
> The latest stable version of RedHat build of OpenTelemetry.

**34. MCOA supports Logs collection and forwarding based on which specific Red Hat OpenShift Logging version?**
> Red Hat OpenShift Logging 6.0.

**35. In the legacy MCO stack (prior to MCOA), what annotation was used on the MultiClusterObservability CR to disable only user workload alert forwarding?**
> `mco-disable-uwl-alerting=true`.

**36. In the legacy MCO stack, setting the annotation `mco-disable-alerting: "true"` on the MultiClusterObservability CR would disable forwarding for which alerts?**
> Both platform and user workload alerts.

### Feature Context and Environment

**37. What new development preview feature in ACM 2.15 provides dashboards to display total timeseries, impactful metrics, and active clusters, helping to manage data proliferation?**
> Cardinality Dashboards.

**38. Enabling Cardinality Dashboards requires modifying what component on the Hub cluster?**
> Thanos, specifically requiring enabling custom rules on Thanos.

**39. What is the name of the Technology Preview feature introduced in ACM 2.15 (ACM-22445) related to VM resource optimization?**
> Virtualization Right-sizing Recommendation with MCO.

**40. The Virtualization Right-Sizing Recommendation provides recommendations based on comparing VM usage against what two specified metrics?**
> CPU & memory usage versus requests.

**41. When configuring Global Hub Observability, what configuration must be disabled on the managed hubs?**
> Hub Self management (`disableHubSelfManagement` set to `true` initially, then rename).

**42. In a Global Hub deployment, how does metrics flow differ between the legacy metrics collection method and MCOA on a Managed Hub?**
> With legacy collection, the Managed Hub sends metrics to itself and the Global Hub; with MCOA, the Managed Hub only sends metrics to the Global Hub (not itself).

**43. In MCOA, when exposing metrics to a third-party tool, what two third-party metric tools are specifically mentioned in the sources as compatible export targets?**
> Kafka and Victoria Metrics.

**44. The MCOA updated Grafana dashboards include which type of new platform monitoring dashboards?**
> Networking dashboards.

**45. What is the configuration hierarchy used to control deployment options (like nodeSelector or proxy settings), where values set here take precedence over direct modifications of other resources (like Prometheus Agents)?**
> `AddonDeploymentConfig`.

**46. What field in the MultiClusterObservability CR allows users to set a desired number of workers for parallel collection in the metrics collector, supporting sharded forwarding?**
> The `workers` field in the MultiClusterObservability CR Spec.

**47. What does the term "Cardinality" refer to in Prometheus and ACM Observability?**
> The total number of unique time series, which is the unique combination of the metric name and all its labels.

**48. For custom metrics collection, the legacy configuration used an `observability-metrics-custom-allowlist` ConfigMap. What specific namespace did this ConfigMap need to be created in on the Hub cluster?**
> `open-cluster-management-observability` namespace.

**49. What key benefit does the long-term storage solution, Thanos, provide in the observability stack?**
> Highly available Prometheus setup with global query view and unlimited retention capabilities.

**50. In ACM 2.14, what key capability related to alerts was added regarding user workload monitoring?**
> Managed cluster user workload alerts are now forwarded to the Hub alert manager, in addition to managed cluster platform alerts.
