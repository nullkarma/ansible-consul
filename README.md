# Consul

Manage HashiCorp Consul (>=1.4.0) with Ansible

## ACL management

This role is capable of bootstrapping your Consul Cluster, manage ACL Policies, Tokens and Token-Policy bindings.
By default Ansible will bootstrap an ACL enabled Cluster with all necessary Policies and Tokens configured to enable DNS Lookups, Read-only UI access and Agent access. See `defaults/main.yml` for more information.
Unlike the `acl.tokens.master` and the anonymous Token, `acl.tokens.agent` and `acl.tokens.default` will **not** be automatically provisioned in the system by simply setting them in your Consul configuration. Therefore a double restart of your Cluster is required. The initial start with `acl.enabled: true` will enable ACL and bind the global policy to your Master Token. This is where additional Tokens and Policies need to be created and `acl.tokens.agent` has to be added to each Consul Server (and Client) config, before you do the second Cluster restart and have your Consul up and running.

With `consul_acl_managed: true` Ansible will Autopilot this bootstrap process and **persist** Tokens in your configs.
You only provide the `acl.tokens.master` in your Servers `consul_config` and omit `acl.tokens.agent` and `acl.tokens.default` in Servers and Clients. After the initial Cluster up, Agent Tokens will be injected into the config and the 2nd restart will be executed.
To inject a Token into your config dynamically, you have to provide a Token-Policy map.

```yaml
consul_token_mapping:
  agent: "Agent Token"
```

`agent` is short for `acl.tokens.agent`.
In the example above, the Token with the Description "Agent Token" will be injected into `acl.tokens.agent`.

### Policies

```yaml
consul_acl_policies:
- name: agents
  description: Agent Token Policy
  policies:
    node_prefix:
      "":
        policy: write
    service_prefix:
      "":
        policy: read
```

### Tokens

```yaml
consul_acl_tokens:
- description: Agent Token
  policies:
  - name: agents
```

# Role Variables

|Parameter|Default|Description|
|---------|-------|-----------|
|consul_version|`1.4.0`||
|consul_base_url|`https://releases.hashicorp.com/consul/{{ consul_version }}`||
|consul_user|`consul`||
|consul_group|`consul`||
|consul_home_dir|`/opt/consul`||
|consul_conf_dir|`/etc/consul`||
|consul_server_group|`consul-servers`|Inventory group in which the Consul Servers are. (Used for dynamic lookup of Servers)|
|consul_manage_service|`true`|Manage Service start and restart|
|consul_restart_on_change|`true`|Service restart on config change|
|consul_acl_managed|`true`|Manage ACL completely with ansible|
|consul_config|see `defaults/main.yml`|JSON config written in yaml. Consult consul documentation for parameters|
|consul_acl_policies|see example for syntax|JSON consul Policies|
|consul_acl_tokens|see example for syntax|JSON consul Tokens|
|consul_token_mapping|see example for syntax|inject Token into config|

# Dependencies

None

# Example Playbook

```yaml
- hosts: consul-servers
  tasks:
  - import_role:
      name: consul
  vars:
    consul_config:
      server: true
      bootstrap_expect: 1
      bind_addr: 0.0.0.0
      datacenter: dc1
      enable_syslog: true
      node_name: "{{ ansible_hostname }}"
      data_dir: /opt/consul/data
```

# License

BSD

# Author Information

see `meta/main.yml`
