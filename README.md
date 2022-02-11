# Ansible Role: prometheus-node-exporter

An Ansible role that installs Prometheus Node Exporter on Ubuntu|Debian|Redhat-based machines with systemd|Upstart|sysvinit.

## Requirements

All needed packages will be installed with this role.

## Role Variables

| Variable  | Type  | Choices |  Default  | Comment  |
|-----------|-------|---------|-----------|----------|
| `prometheus_node_exporter_version` | string | See [node_exporter](https://github.com/prometheus/node_exporter/releases) releases | 0.18.1    | Version of node_exporter that will be installed. Minimal supported version: 0.15    |
| `prometheus_node_exporter_release_name`       | string |  | `node_exporter-{{ prometheus_node_exporter_version }}.linux-amd64`   | Name of the binary that will be downloaded from the [release](https://github.com/prometheus/node_exporter/releases) page |
| `prometheus_node_exporter_enabled_collectors` | list   | [List of flags](https://github.com/prometheus/node_exporter#disabled-by-default) | `[]`| List of [collectors that are disabled by default](https://github.com/prometheus/node_exporter#disabled-by-default) to enable   |
| `prometheus_node_exporter_disabled_collectors` | list   | [List of flags](https://github.com/prometheus/node_exporter#enabled-by-default)| `[]`| List of [collectors that are enabled by default](https://github.com/prometheus/node_exporter#enabled-by-default) to disable  |
| `prometheus_node_exporter_config_flags`  | dict   |   |  | Dict of key, value options to add to the start command line  |
| `prometheus_node_exporter_url`    | string |      | not defined   | Custom URL to download node_exporter if you don't have access to GitHub     |
| `prometheus_node_exporter_tls_cert` | string | See [TLS and authentication](https://prometheus.io/docs/prometheus/latest/configuration/https/) | not defined | PEM formatted X.509 certificate  |
| `prometheus_node_exporter_tls_key` | string |See [TLS and authentication](https://prometheus.io/docs/prometheus/latest/configuration/https/) | not defined | PEM formatted X.509 private key  |
| `prometheus_node_exporter_tls_client_ca` | string |See [TLS and authentication](https://prometheus.io/docs/prometheus/latest/configuration/https/) | not defined | PEM formatted X.509 client CA |
| `prometheus_node_exporter_basic_auth_users` | dict |See [TLS and authentication](https://prometheus.io/docs/prometheus/latest/configuration/https/) | `{}` | Key/value pairs representing usernames and (cleartext) passwords - the role takes care of hashing the passwords.  |
| `prometheus_node_exporter_http_server_config` | dict |See [TLS and authentication](https://prometheus.io/docs/prometheus/latest/configuration/https/) | `{}` | Web server options  |

## Dependencies

- UnderGreen.prometheus-exporters-common

## Example Playbooks

```yaml
- hosts: node-exporters
  roles:
    - role: undergreen.prometheus-node-exporter
      prometheus_node_exporter_version: 0.18.1
      prometheus_node_exporter_enabled_collectors:
        - conntrack
        - cpu
        - diskstats
        - entropy
        - filefd
        - filesystem
        - loadavg
        - mdadm
        - meminfo
        - netdev
        - netstat
        - stat
        - textfile
        - time
        - vmstat
      prometheus_node_exporter_config_flags:
        'web.listen-address': '0.0.0.0:9100'
        'log.level': 'info'
```

### TLS using an X.509 key pair that you provide, plus basic auth
```yaml
- hosts: secure-node-exporters
  roles:
    - role: undergreen.prometheus-node-exporter
      prometheus_node_exporter_version: 1.3.1
      prometheus_node_exporter_tls_cert: |
        -----BEGIN CERTIFICATE-----
        MIIBgDCCASWgAwIBAgIUaa+4EIvsObFzij6Ntc6SFXKQoKkwCgYIKoZIzj0EAwIw
        FTETMBEGA1UEAwwKcHJvbWV0aGV1czAeFw0yMjAyMTAxOTIxMTBaFw0zODAxMTcx
        OTIxMTBaMBUxEzARBgNVBAMMCnByb21ldGhldXMwWTATBgcqhkjOPQIBBggqhkjO
        PQMBBwNCAASbr9rMFdAl+wQp48sLzzG+fqPMY5jgD4mKFZfQ4kEt32/9WBtJ8GhJ
        Fkd1EwSwWM/4Lr06QxPX2HOYKtT9OTLEo1MwUTAdBgNVHQ4EFgQUijFndd4tyCbI
        3LOpvYw5Hv6w/JEwHwYDVR0jBBgwFoAUijFndd4tyCbI3LOpvYw5Hv6w/JEwDwYD
        VR0TAQH/BAUwAwEB/zAKBggqhkjOPQQDAgNJADBGAiEAxhxxDGsv5Im6EPtdEqo4
        Al40BgVMUVNuKW0aL1cPAT0CIQDPeqgPBXgzezSj5/ZXu+jAUCACsgMD3GrfusQj
        HBXKVw==
        -----END CERTIFICATE-----
      prometheus_node_exporter_tls_key: "{{ lookup('file','keys/proj/tls1.key') }}"
      prometheus_node_exporter_basic_auth_users:
        admin: hackme
        foo: bar
```

### TLS using an auto genereated self signed key pair, no authentication
```yaml
- hosts: secure-node-exporters
  roles:
    - role: undergreen.prometheus-node-exporter
      prometheus_node_exporter_version: 1.3.1
      prometheus_node_exporter_tls_selfsigned: true
```


### TLS using an auto generated key pair with custom options, signed with a CA that you own, plus basic auth
```yaml
- hosts: secure-node-exporters
  roles:
    - role: undergreen.prometheus-node-exporter
      prometheus_node_exporter_version: 1.3.1
      prometheus_node_exporter_tls_ownca: true
      prometheus_node_exporter_tls_ownca_content: |
        -----BEGIN CERTIFICATE-----
        MIICMTCCAdegAwIBAgIUV7fDRGKIW37zkonNR6U/5ecD6pswCgYIKoZIzj0EAwQw
        ZjELMAkGA1UEBhMCTkwxDzANBgNVBAoMBkfDiUFOVDEXMBUGA1UECwwOSy5XYXJr
        dGFhcnRqZXMxLTArBgNVBAMMJEsuV2Fya3RhYXJ0amVzIENlcnRpZmljYXRlIEF1
        dGhvcml0eTAeFw0yMjAyMTExNDU1MjdaFw0zODAxMTkwMzE0MDdaMGYxCzAJBgNV
        BAYTAk5MMQ8wDQYDVQQKDAZHw4lBTlQxFzAVBgNVBAsMDksuV2Fya3RhYXJ0amVz
        MS0wKwYDVQQDDCRLLldhcmt0YWFydGplcyBDZXJ0aWZpY2F0ZSBBdXRob3JpdHkw
        WTATBgcqhkjOPQIBBggqhkjOPQMBBwNCAAQftoqmBvFN0vJRfa5M878vuUs4bBBh
        54UCPJwhFdrrlXMZl79ZjDiT++tmIKdCbo8YVhKxc1lPTP+TyJQl2ZsSo2MwYTAv
        BgNVHREEKDAmgiRLLldhcmt0YWFydGplcyBDZXJ0aWZpY2F0ZSBBdXRob3JpdHkw
        DwYDVR0TAQH/BAUwAwEB/zAdBgNVHQ4EFgQUF1gsr+lXy6dWyUofZqeehTxtuEMw
        CgYIKoZIzj0EAwQDSAAwRQIgOwqrnfEAE8UIgd7IonHlyDHYnkPxubLF/YEasCEU
        K/MCIQDXCNpcgNzenDM5AWUg2giyss8r1h58kiO7SrEZ0pXR0A==
        -----END CERTIFICATE-----
      prometheus_node_exporter_tls_ownca_privatekey_content: "{{ lookup('file', '/opt/ca/very/secret.key') }}"
      prometheus_node_exporter_tls_key_type: ECC
      prometheus_node_exporter_tls_key_curve: secp256r1
      prometheus_node_exporter_tls_cert_digest: sha512
      prometheus_node_exporter_basic_auth_users:
        admin: hackme
        foo: bar
```


## Note:

Due to [prometheus/node_exporter#640](https://github.com/prometheus/node_exporter/pull/640) and [prometheus/node_exporter#639](https://github.com/prometheus/node_exporter/pull/639) changes, this role can only support the minimum version 0.15 of node_exporter.

## License

GPLv2
