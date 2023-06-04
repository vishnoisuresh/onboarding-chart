# Onboarding Chart

This Helm chart deploys resources for multiple teams in a cluster. Each team can have different configurations for network policies, resource quotas, and role bindings.
## Prerequisites

- Kubernetes/OpenShift cluster
- Helm
## Team Configuration

Teams are defined in the `values.yaml` file. Each team has the following properties:

- `name`: The name of the team.
- `namespaces`: A list with the namespaces to be generated.
- `networkPolicy`: The configuration for network policies. It has the following properties:
  - `enabled`: A boolean value indicating whether network policies should be created or not. [Read more here](#network-policies)
- `resourceQuota`: The configuration for resource quotas. It has the following properties:
  - `enabled`: A boolean value indicating whether resource quotas should be created for the team. When set to `true`, resource quotas will be created. When set to `false`, no resource quotas will be created.
  - `cpu`: The CPU limit for the team's resource quota.
  - `memory`: The memory limit for the team's resource quota.
- `projectRole`: The role assigned to the team in the cluster. If not specified, the default role is `admin`.

```bash
| Parameter                               | Description                                  | Default |
|-----------------------------------------|----------------------------------------------|---------|
| teams                                   | List of teams to create                      |   []    |
| teams.name                              | Name of the team                             |         |
| teams.namespaces                        | List of the namespaces to be created         |   []    |
| teams.namespaces.name                   | Name of the namespace to be created          |         |
| teams.namespaces.networkPolicy          | Network policy configuration for the team    |         |
| teams.namespaces.networkPolicy.enabled  | True/False                                   |  False  |
| teams.namespaces.resourceQuota          | Resource quota configuration for the team    |         |
| teams.namespaces.resourceQuota.enabled  | True/False                                   |  False  |
| teams.namespaces.resourceQuota.cpu      | CPU limit for the team                       |         |
| teams.namespaces.resourceQuota.memory   | Memory limit for the team                    |         |
| teams.namespaces.projectRole            | The role assigned to the team in the cluster |  admin  |
```

## Example Usage
To create a Teams Chart deployment with custom values, create a values.yaml file with your desired configuration:

```yaml
teams:
# Team 1
  - name: team1                     <--- The name of the team
    namespaces:
      - name: team-1-namespace      <--- Name of the namespace being created
        projectRole: admin          <--- ClusterRole assigned to the team. Default = admin   
        networkPolicy:              <--- Enabled or not. Creates default networkPolicies. Default = false
          enabled: true
        resourceQuota:              <--- Enabled or not. Creates resourceQuotas for the namespace. Default = false
          enabled: true
          cpu: "4"
          memory: 8Gi
# Team 2
  - name: team2
    namespaces:
      - name: team-2-namespace
        projectRole: admin
        networkPolicy:
          enabled: true
        resourceQuota:
          enabled: true
          cpu: "2"
          memory: 6Gi

      - name: team-2-namespace-2
        projectRole: edit
        networkPolicy:
          enabled: true
        resourceQuota:
          enabled: false
```
## Created Resources

The following resources will be created for each team based on their configuration:

### Namespace

One or more namespaces will be created for each team. The namespace name will be based on the team's `namespaces.name` property.

### Network Policies

If the `networkPolicy` is set to `enabled` for a namespace, the following network policies will be created:

- A network policy named `<team-name>-deny-by-default` in the selected namespace. This policy denies all incoming traffic by default.
- A network policy named `<team-name>-allow-same-namespace` in the selected namespace. This policy allows incoming traffic from any pod within the same namespace.

[More information](https://docs.openshift.com/container-platform/latest/networking/network_policy/creating-network-policy.html#nw-networkpolicy-create-cli_creating-network-policy)
### Resource Quotas

If the `resourceQuota.enabled` is set to `true` for a namespace, a resource quota will be created in the selected namespace. The resource quota will have the CPU and memory limits specified in the namespace configuration.

### Role Bindings

A role binding will be created for each namespace, assigning the team a role in the cluster. If the `projectRole` is not specified for a namespace, the default role assigned is `admin`. The role binding name will be `<team-name>-<project-role>-rolebinding`.