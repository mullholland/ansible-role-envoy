# [envoy](#envoy)

Installs an configures envoy

|GitHub|GitLab|Quality|Downloads|Version|
|------|------|-------|---------|-------|
|[![github](https://github.com/mullholland/ansible-role-envoy/workflows/Ansible%20Molecule/badge.svg)](https://github.com/mullholland/ansible-role-envoy/actions)|[![gitlab](https://gitlab.com/opensourceunicorn/ansible-role-envoy/badges/master/pipeline.svg)](https://gitlab.com/opensourceunicorn/ansible-role-envoy)|[![quality](https://img.shields.io/ansible/quality/59701)](https://galaxy.ansible.com/mullholland/envoy)|[![downloads](https://img.shields.io/ansible/role/d/59701)](https://galaxy.ansible.com/mullholland/envoy)|[![Version](https://img.shields.io/github/release/mullholland/ansible-role-envoy.svg)](https://github.com/mullholland/ansible-role-envoy/releases/)|

## [Example Playbook](#example-playbook)

This example is taken from [`molecule/default/converge.yml`](https://github.com/mullholland/ansible-role-envoy/blob/master/molecule/default/converge.yml) and is tested on each push, pull request and release.

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

The machine needs to be prepared. In CI this is done using [`molecule/default/prepare.yml`](https://github.com/mullholland/ansible-role-envoy/blob/master/molecule/default/prepare.yml):

```yaml
---
- name: Prepare
  hosts: all
  become: true
  gather_facts: true

  tasks:
    - name: Debian | Install xz-utils
      ansible.builtin.package:
        name: "xz-utils"
        state: present
      when: ansible_os_family == "Debian"

    - name: RedHat | Install xz
      ansible.builtin.package:
        name:
          - "xz"
          - "tar"
        state: present
      when: ansible_os_family == "RedHat"
```


## [Role Variables](#role-variables)

The default values for the variables are set in [`defaults/main.yml`](https://github.com/mullholland/ansible-role-envoy/blob/master/defaults/main.yml):

```yaml
---
envoy_bin_path: "/usr/local/bin"

# Disabled if installed as sidecar servie for example consul usage
# https://learn.hashicorp.com/tutorials/consul/consul-terraform-sync-intro?in=consul/network-infrastructure-automation#install-envoy-optional
envoy_conf_path: "/etc/envoy"

envoy_user: "envoy"
envoy_group: "envoy"
# wether envoy user should be a system user or not
envoy_system_user: true

# Download
# https://archive.tetratelabs.io/envoy/envoy-versions.json
envoy_version: "1.17.0"
envoy_architecture_map:
  amd64: amd64
  x86_64: amd64
  armv7l: arm
  aarch64: arm64
  32-bit: "386"
  64-bit: amd64
envoy_arch: "linux-{{ envoy_architecture_map[ansible_architecture] }}"
# optional
# clear if unused
# envoy_checksum: "sha256:fbd2460189f330a6b1e6b4ff79b4604dff1847091d4abbdda2c6b3894fadb396"
envoy_download_url: "https://archive.tetratelabs.io/envoy/download/v{{ envoy_version }}/envoy-v{{ envoy_version }}-{{ envoy_arch }}.tar.xz"
envoy_download_path: "/tmp/envoy-v{{ envoy_version }}-{{ envoy_arch }}.tar.xz"
envoy_unarchive_dest: "/tmp"
envoy_unarchive_creates: "/tmp/envoy-v{{ envoy_version }}-{{ envoy_arch }}/bin/envoy"

# Create a systemd servicefile
envoy_systemd: false

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

## [Requirements](#requirements)

- pip packages listed in [requirements.txt](https://github.com/mullholland/ansible-role-envoy/blob/master/requirements.txt).


## [Context](#context)

This role is a part of many compatible roles. Have a look at [the documentation of these roles](https://mullholland.net) for further information.

Here is an overview of related roles:
![dependencies](https://raw.githubusercontent.com/mullholland/ansible-role-envoy/png/requirements.png "Dependencies")

## [Compatibility](#compatibility)

This role has been tested on these [container images](https://hub.docker.com/u/mullholland):

|container|tags|
|---------|----|
|[EL](https://hub.docker.com/repository/docker/mullholland/docker-centos-systemd/general)|8, 9|
|[Amazon](https://hub.docker.com/repository/docker/mullholland/docker-amazonlinux-systemd/general)|Candidate|
|[Fedora](https://hub.docker.com/repository/docker/mullholland/docker-fedora-systemd/general)|all|
|[Ubuntu](https://hub.docker.com/repository/docker/mullholland/docker-ubuntu-systemd/general)|all|
|[Debian](https://hub.docker.com/repository/docker/mullholland/docker-debian-systemd/general)|all|

The minimum version of Ansible required is 2.10, tests have been done to:

- The previous version.
- The current version.
- The development version.

If you find issues, please register them in [GitHub](https://github.com/mullholland/ansible-role-envoy/issues)

## [License](#license)

[MIT](https://github.com/mullholland/ansible-role-envoy/blob/master/LICENSE).

## [Author Information](#author-information)

[Mullholland](https://mullholland.net)

Please consider [sponsoring me](https://github.com/sponsors/mullholland).
