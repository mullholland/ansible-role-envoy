# [envoy](#envoy)

|GitHub|GitLab|
|------|------|
|[![github](https://github.com/mullholland/ansible-role-envoy/workflows/Ansible%20Molecule/badge.svg)](https://github.com/mullholland/ansible-role-envoy/actions)|[![gitlab](https://gitlab.com/mullholland/ansible-role-envoy/badges/master/pipeline.svg)](https://gitlab.com/mullholland/ansible-role-envoy)|[![quality](https://img.shields.io/ansible/quality/unset)](https://galaxy.ansible.com/mullholland/envoy)|

description

## [Role Variables](#role-variables)

These variables are set in `defaults/main.yml`:
```yaml
---
envoy_bin_path: "/usr/local/bin"

# Disabled if installed as sidecar servie for example consul usage
# https://learn.hashicorp.com/tutorials/consul/consul-terraform-sync-intro?in=consul/network-infrastructure-automation#install-envoy-optional
envoy_enable_conf: false
envoy_conf_path: "/etc/envoy"

envoy_user: "envoy"
envoy_group: "envoy"
# wether envoy user should be a system user or not
envoy_system_user: true

# Download
# https://archive.tetratelabs.io/envoy/envoy-versions.json
envoy_version: "1.22.2"
envoy_arch: "linux-amd64"
# optional
# clear if unused
envoy_checksum: "sha256:fbd2460189f330a6b1e6b4ff79b4604dff1847091d4abbdda2c6b3894fadb396"
envoy_download_url: "https://archive.tetratelabs.io/envoy/download/v{{ envoy_version }}/envoy-v{{ envoy_version }}-{{ envoy_arch }}.tar.xz"
envoy_download_path: "/tmp/envoy-v{{ envoy_version }}-{{ envoy_arch }}.tar.xz"
envoy_unarchive_dest: "/tmp"
envoy_unarchive_creates: "/tmp/envoy-v{{ envoy_version }}-{{ envoy_arch }}/bin/envoy"

# Create a systemd servicefile
envoy_systemd: false
envoy_systemd_enabled: true

# will be used with `to_nice_yaml`
envoy_config: []
# admin:
#   address:
#     socket_address: { address: 127.0.0.1, port_value: 9901 }
# static_resources:
#   listeners:
#   - name: listener_0
#     address:
#       socket_address: { address: 127.0.0.1, port_value: 10000 }
#     filter_chains:
#     - filters:
#       - name: envoy.filters.network.http_connection_manager
#         typed_config:
#           "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
#           stat_prefix: ingress_http
#           codec_type: AUTO
#           route_config:
#             name: local_route
#             virtual_hosts:
#             - name: local_service
#               domains: ["*"]
#               routes:
#               - match: { prefix: "/" }
#                 route: { cluster: some_service }
#           http_filters:
#           - name: envoy.filters.http.router
#             typed_config:
#               "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router
#   clusters:
#   - name: some_service
#     connect_timeout: 0.25s
#     type: STATIC
#     lb_policy: ROUND_ROBIN
#     load_assignment:
#       cluster_name: some_service
#       endpoints:
#       - lb_endpoints:
#         - endpoint:
#             address:
#               socket_address:
#                 address: 127.0.0.1
#                 port_value: 1234
```


## [Example Playbook](#example-playbook)

This example is taken from `molecule/default/converge.yml` and is tested on each push, pull request and release.
```yaml
---
- name: Converge
  hosts: all
  become: true
  gather_facts: true
  vars:
    envoy_systemd: true
    envoy_config:
      admin:
        address:
          socket_address: {address: 127.0.0.1, port_value: 9901}
      static_resources:
        listeners:
          - name: listener_0
            address:
              socket_address: {address: 127.0.0.1, port_value: 10000}
        filter_chains:
          - filters:
              - name: envoy.filters.network.http_connection_manager
                typed_config:
                  "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
                stat_prefix: ingress_http
                codec_type: AUTO
                route_config:
                  name: local_route
                virtual_hosts:
                  - name: local_service
                    domains: ["*"]
                    routes:
                      - match: {prefix: "/"}
                        route: {cluster: some_service}
                http_filters:
                  - name: envoy.filters.http.router
                    typed_config:
                      "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router
        clusters:
          - name: some_service
            connect_timeout: 0.25s
            type: STATIC
            lb_policy: ROUND_ROBIN
            load_assignment:
              cluster_name: some_service
              endpoints:
                - lb_endpoints:
                    - endpoint:
                        address:
                          socket_address:
                            address: 127.0.0.1
                            port_value: 1234

  roles:
    - role: "mullholland.envoy"
```

The machine needs to be prepared in CI this is done using `molecule/default/prepare.yml`:
```yaml
---
- name: Prepare
  hosts: all
  become: true
  gather_facts: true

  tasks:
    - name: install xz-utils
      ansible.builtin.package:
        name: "xz-utils"
        state: present

```





## [Compatibility](#compatibility)

This role has been tested on these [container images](https://hub.docker.com/u/mullholland):

-   [debian10](https://hub.docker.com/r/mullholland/docker-molecule-debian10)
-   [debian11](https://hub.docker.com/r/mullholland/docker-molecule-debian11)
-   [ubuntu1804](https://hub.docker.com/r/mullholland/docker-molecule-ubuntu1804)
-   [ubuntu2004](https://hub.docker.com/r/mullholland/docker-molecule-ubuntu2004)
-   [ubuntu2204](https://hub.docker.com/r/mullholland/docker-molecule-ubuntu2204)
-   [centos7](https://hub.docker.com/r/mullholland/docker-molecule-centos7)
-   [centos-stream8](https://hub.docker.com/r/mullholland/docker-molecule-centos-stream8)
-   [fedora35](https://hub.docker.com/r/mullholland/docker-molecule-fedora35)
-   [fedora36](https://hub.docker.com/r/mullholland/docker-molecule-fedora36)
-   [amazonlinux](https://hub.docker.com/r/mullholland/docker-molecule-amazonlinux)
-   [rockylinux8](https://hub.docker.com/r/mullholland/docker-molecule-rockylinux8)
-   [almalinux8](https://hub.docker.com/r/mullholland/docker-molecule-almalinux8)

The minimum version of Ansible required is 2.10, tests have been done to:

-   The previous versions.
-   The current version.
-   The [devel](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-devel-from-github-with-pip) version.





If you find issues, please register them in [GitHub](https://github.com/mullholland/ansible-role-envoy/issues)

## [License](#license)

MIT


## [Author Information](#author-information)

[Mullholland](https://github.com/mullholland)

## [Special Thanks](#special-thanks)

Template inspired by [Robert de Bock](https://github.com/robertdebock)
