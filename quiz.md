Here is the MCOA Deep Dive Quiz formatted in Markdown. You can copy the text below directly into any Markdown editor (like Obsidian, GitHub, or VS Code).

MCOA Deep Dive Quiz (50 Questions)
Core Identity, Purpose, and Status
1. What is the primary function of the MultiCluster Observability (MCO) component within RHACM?

Answer: MCO allows users to gain insights and optimize RHACM managed clusters.

2. What is the MultiCluster Observability Addon (MCOA) described as?

Answer: MCOA is an evolution of the MultiCluster Observability Operator, focusing on providing a more efficient metrics collection architecture for multicluster environments.

3. What are the three primary observability signals (data types) that MCOA is designed to orchestrate and include collection, storage, authorization (authz), and UI for?

Answer: Metrics, Logs, and Traces.

4. In which major RHACM version was the Multi-cluster Observability Addon (MCOA) for Metrics listed as being generally available (GA)?

Answer: ACM 2.15.

5. In which RHACM version was the MCOA component initially delivered, hidden behind a feature flag?

Answer: ACM 2.12.

6. MCOA re-uses the APIs created for what type of user stories?

Answer: Single OpenShift Container Platform (OCP) user stories.

7. When enabled for metrics collection, what older components does MCOA replace on the managed cluster side?

Answer: The Endpoint operator and the Metrics Collector.

8. What key element of the existing observability stack remains unchanged when MCOA is enabled for metrics collection?

Answer: Thanos storage.

Benefits and Improvements
9. What is the main benefit MCOA provides regarding configurability compared to the previous metrics stack?

Answer: Improved configurability by enabling users to directly configure custom resources deployed on spokes using standard upstream Prometheus operator APIs.

10. How does MCOA achieve increased metrics federation scalability?

Answer: Through sharded metrics federation using distinct ScrapeConfigs, each independently federated from the in-cluster Prometheus.

11. What industry-standard implementation does MCOA use to send metrics efficiently from spokes to the hub?

Answer: Standard remoteWrite implementation.

12. MCOA's architecture provides resiliency to network partitions between the spoke and hub for how long without data loss?

Answer: Up to 2 hours.

13. What improvement related to metrics collection ensures there are "No More Gaps" in data, providing continuous visibility?

Answer: Full-Time Pod Granularity, ensuring all dashboards continuously display metrics at the individual pod level (24/7), resolving previous issues where critical pod-level metrics were only available during alerts.

14. What are MCOA's default platform metrics optimized and curated for?

Answer: Performance, while maintaining pod-level observability and minimizing cardinality.

Architecture and Components
15. What is the new metrics collector workload running on managed clusters (spokes) that replaces the legacy Metrics Collector?

Answer: The Prometheus Agent.

16. What configuration API resources are used in MCOA to define metrics collection configuration, replacing the legacy Allow list ConfigMap?

Answer: ScrapeConfig CRs and PrometheusRule CRs.

17. On the hub cluster, what new component is deployed when enabling MCOA?

Answer: The multicluster-observability-addon-manager.

18. What API is used by MCOA to define precisely what configuration is dedicated to what cluster, acting as a multicluster scheduler?

Answer: The Placement API from Open Cluster Management (OCM).

19. Which configuration API in MCOA specifies options like the installation namespace or node selector for the deployed addon?

Answer: AddonDeploymentConfig.

20. What is the default installation namespace for MCOA resources deployed on managed clusters (spokes)?

Answer: open-cluster-management-agent-addon.

21. For non-OCP managed clusters, MCOA supports the deployment of which three additional components compared to OCP clusters?

Answer: The Node Exporter, Kube State Metrics, and a Prometheus server.

Configuration and Management
22. When enabling MCOA in the MultiClusterObservability CR, which specific metric type is noted as being required to be enabled?

Answer: Platform metrics (e.g., spec.capabilities.platform.metrics.default.enabled: true).

23. When adding custom metrics via the MCOA configuration, what specific resource type replaces the use of the legacy metrics_list.yaml?

Answer: ScrapeConfig.

24. When adding custom recording rules within MCOA, which Kubernetes Custom Resource Definition (CRD) should be used?

Answer: PrometheusRule (specifically from the monitoring.coreos.com group).

25. What operation does MCOA use to track and automatically revert modifications if a user attempts to change an enforced configuration field in a Prometheus Agent?

Answer: Server-side apply.

26. In the structure used for migrating custom metric allow lists (Names, Matches, Recording_rules, Collect_rules, Renames), which two capabilities are explicitly stated as "not supported" in MCOA?

Answer: Renames and Collect_rules.

27. If a ScrapeConfig for user workload metrics is deployed in a specific namespace on the spoke cluster, what restriction is enforced on the metrics collected?

Answer: It will only collect metrics coming from that specific namespace, enforcing the namespace label on the collected metrics.

28. To limit metrics collection to spokes running Single Node OpenShift (SNO), what mechanism, using placement predicates, can be used to identify these clusters?

Answer: Selecting ManagedClusters with the label vendor: OpenShift and the claim with name controlplanetopology.openshift.io and value SingleReplica.

29. If a managed cluster is associated with multiple placements referenced in the ClusterManagementAddon, whose configuration will the cluster inherit?

Answer: The configuration from the last compatible placement listed in the ClusterManagementAddon.

30. To entirely disable metrics collection on a specific managed cluster, what configuration action is required regarding placements?

Answer: The cluster must be excluded from all placements referenced in the ClusterManagementAddOn.

Alerting and Logs
31. By default, what is the policy regarding alert forwarding from OCP managed clusters to the Hub Alertmanager when using MCOA?

Answer: By default, MCOA does not configure alert forwarding automatically.

32. What method is recommended for enabling alert forwarding in MCOA, overcoming the default disabled behavior?

Answer: Using an RHACM Policy (specifically a ConfigurationPolicy applied via Policy).

33. MCOA supports Traces collection and forwarding based on which technology/build?

Answer: The latest stable version of RedHat build of OpenTelemetry.

34. MCOA supports Logs collection and forwarding based on which specific Red Hat OpenShift Logging version?

Answer: Red Hat OpenShift Logging 6.0.

35. In the legacy MCO stack (prior to MCOA), what annotation was used on the MultiClusterObservability CR to disable only user workload alert forwarding?

Answer: mco-disable-uwl-alerting=true.

36. In the legacy MCO stack, setting the annotation mco-disable-alerting: "true" on the MultiClusterObservability CR would disable forwarding for which alerts?

Answer: Both platform and user workload alerts.

Feature Context and Environment
37. What new development preview feature in ACM 2.15 provides dashboards to display total timeseries, impactful metrics, and active clusters, helping to manage data proliferation?

Answer: Cardinality Dashboards.

38. Enabling Cardinality Dashboards requires modifying what component on the Hub cluster?

Answer: Thanos, specifically requiring enabling custom rules on Thanos.

39. What is the name of the Technology Preview feature introduced in ACM 2.15 (ACM-22445) related to VM resource optimization?

Answer: Virtualization Right-sizing Recommendation with MCO.

40. The Virtualization Right-Sizing Recommendation provides recommendations based on comparing VM usage against what two specified metrics?

Answer: CPU & memory usage versus requests.

41. When configuring Global Hub Observability, what configuration must be disabled on the managed hubs?

Answer: Hub Self management (disableHubSelfManagement set to true initially, then rename).

42. In a Global Hub deployment, how does metrics flow differ between the legacy metrics collection method and MCOA on a Managed Hub?

Answer: With legacy collection, the Managed Hub sends metrics to itself and the Global Hub; with MCOA, the Managed Hub only sends metrics to the Global Hub (not itself).

43. In MCOA, when exposing metrics to a third-party tool, what two third-party metric tools are specifically mentioned in the sources as compatible export targets?

Answer: Kafka and Victoria Metrics.

44. The MCOA updated Grafana dashboards include which type of new platform monitoring dashboards?

Answer: Networking dashboards.

45. What is the configuration hierarchy used to control deployment options (like nodeSelector or proxy settings), where values set here take precedence over direct modifications of other resources (like Prometheus Agents)?

Answer: AddonDeploymentConfig.

46. What field in the MultiClusterObservability CR allows users to set a desired number of workers for parallel collection in the metrics collector, supporting sharded forwarding?

Answer: The workers field in the MultiClusterObservability CR Spec.

47. What does the term "Cardinality" refer to in Prometheus and ACM Observability?

Answer: The total number of unique time series, which is the unique combination of the metric name and all its labels.

48. For custom metrics collection, the legacy configuration used an observability-metrics-custom-allowlist ConfigMap. What specific namespace did this ConfigMap need to be created in on the Hub cluster?

Answer: open-cluster-management-observability namespace.

49. What key benefit does the long-term storage solution, Thanos, provide in the observability stack?

Answer: Highly available Prometheus setup with global query view and unlimited retention capabilities.

50. In ACM 2.14, what key capability related to alerts was added regarding user workload monitoring?

Answer: Managed cluster user workload alerts are now forwarded to the Hub alert manager, in addition to managed cluster platform alerts.
