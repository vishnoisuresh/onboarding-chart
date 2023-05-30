# Onboarding Chart

This Helm chart deploys resources for multiple teams in a cluster. Each team can have different configurations for network policies, resource quotas, and role bindings.
## Prerequisites

- Kubernetes/OpenShift cluster
- Helm
## Team Configuration

Teams are defined in the `values.yaml` file. Each team has the following properties:

- `name`: The name of the team.
- `namespace`: The name of the namespace to be generated.
- `networkPolicy`: The configuration for network policies. It can be set to `enabled` or `disabled`. When set to `enabled`, network policies will be created for the team. When set to `disabled`, no network policies will be created.
- `resourceQuota`: The configuration for resource quotas. It has the following properties:
  - `enabled`: A boolean value indicating whether resource quotas should be created for the team. When set to `true`, resource quotas will be created. When set to `false`, no resource quotas will be created.
  - `cpu`: The CPU limit for the team's resource quota.
  - `memory`: The memory limit for the team's resource quota.
- `projectRole`: The role assigned to the team in the cluster. If not specified, the default role is `admin`.

```bash
| Parameter                   | Description                                  | Default |
|-----------------------------|----------------------------------------------|---------|
| teams                       | List of teams to create                      |   []    |
| teams.name                  | Name of the team                             |         |
| teams.namespace             | Name of the namespace to be created          |         |
| teams.networkPolicy         | Network policy configuration for the team    |         |
| teams.networkPolicy.enabled | True/False                                   | False   |
| teams.resourceQuota         | Resource quota configuration for the team    |         |
| teams.resourceQuota.enabled | True/False                                   | False   |
| teams.resourceQuota.cpu     | CPU limit for the team                       |         |
| teams.resourceQuota.memory  | Memory limit for the team                    |         |
| teams.projectRole           | The role assigned to the team in the cluster | admin   |
```

## Example Usage
To create a Teams Chart deployment with custom values, create a values.yaml file with your desired configuration:

```yaml
teams:
  - name: team1                  < The name of the team
    namespace: team-1-namespace  < Name of the namespace being created
    networkPolicy:               < Enabled or not. Creates default networkPolicies. Default = false
      enabled: true
    resourceQuota:               < Enabled or not. Creates resourceQuotas for the namespace. Default = false
      enabled: true
      cpu: "4"
      memory: 8Gi
    projectRole: admin           < ClusterRole assigned to the team. Default = admin
  - name: team2
    namespace: team-2-namespace
    networkPolicy:
      enabled: false
    resourceQuota:
      enabled: false
```
## Created Resources

The following resources will be created for each team based on their configuration:

### Namespace

A namespace will be created for each team. The namespace name will be based on the team's `namespace` property.

### Network Policies

If the `networkPolicy` is set to `enabled` for a team, the following network policies will be created:

- A network policy named `<team-name>-deny-by-default` in the team's namespace. This policy denies all incoming traffic by default.
- A network policy named `<team-name>-allow-same-namespace` in the team's namespace. This policy allows incoming traffic from any pod within the same namespace.

[More information](https://docs.openshift.com/container-platform/latest/networking/network_policy/creating-network-policy.html#nw-networkpolicy-create-cli_creating-network-policy)
### Resource Quotas

If the `resourceQuota.enabled` is set to `true` for a team, a resource quota will be created in the team's namespace. The resource quota will have the CPU and memory limits specified in the team's configuration.

### Role Bindings

A role binding will be created for each team, assigning them a role in the cluster. If the `projectRole` is not specified for a team, the default role assigned is `admin`. The role binding name will be `<team-name>-<project-role>-rolebinding`.
