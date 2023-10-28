ansible-role-k3s-charts
=======================

This role installs lab Helm charts on K3s.

Requirements
------------

None.

Role Variables
--------------

All default variables are set in `defaults/main.yml`. To change K3s manifest directory use following variable:

```yaml
k3s:
  manifest_dir: /var/lib/rancher/k3s/server/manifests
```

To define Helm charts to be installed, add an item under `charts` list:

```yaml
charts:
  - name: traefik
    namespace: kube-system
```

Most charts use ingress with TLS. Cert-manager is installed to provide valid certificates. To generate certificates, cert-manager requires `dns_zone`, `api_token` and `email`:

```yaml
charts:
  - name: cert-manager
    namespace: cert-manager
    repo: https://charts.jetstack.io
    secret_name: cloudflare-api-token-secret
    dns_zone: "{{ dns_zone }}"
    api_token: "{{ api_token }}"
    email: "{{ email }}"
    nameserver: '1.1.1.1'
```

Prometheus requires list of `endpoints` for exporters. This can be passed by playbook (see example). Also, Grafana has default credentials defined which can be overridden with `grafana_user` and `grafana_password`:

```yaml
charts:
  - name: kube-prometheus-stack
    namespace: monitoring
    repo: https://prometheus-community.github.io/helm-charts
    dns_zone: "{{ dns_zone }}"
    endpoints: "{{ endpoints }}"
    grafana:
      user: "{{ grafana_user | default('admin', true) | b64encode }}"
      password: "{{ grafana_password | default('admin@grafana', true) | b64encode }}"
```

Unifi chart requires list of Master node IPs passed from playbook as `external_ips`:

```yaml
charts:
  - name: unifi
    namespace: unifi
    repo: https://raw.githubusercontent.com/frennky/lab/charts
    version: '0.3.2'
    dns_zone: "{{ dns_zone }}"
    cluster_issuer: "{{ cluster_issuer }}"
    external_ips: "{{ external_ips }}"
```

Dependencies
------------

None.

Example Playbook
----------------

Here is an example playbook that includes required variables by role:

```yaml
---

- name: Install Helm Charts on K3s cluster
  hosts: red
  become: true
  remote_user: pi
  vars:
    external_ips: "{{ groups['server'] | map('extract', hostvars, 'ansible_host') | list }}"
    endpoints: "{{ groups['all'] | map('extract', hostvars, 'ansible_host') | list }}"
    dns_zone: my.net
    api_token: xxxxxxxxxxxxxxxxx
    email: me@my.net
    grafana_user: admin
    grafana_password: admin@grafana
  roles:
    - role: ansible-role-k3s-charts
```

License
-------

MIT

Author Information
------------------

Created by [frennky](https://github.com/frennky).
