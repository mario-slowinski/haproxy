haproxy
=======

Ansible role to manage [HAproxy](http://cbonte.github.io/haproxy-dconv/2.4/configuration.html).

HAproxy configuration in `haproxy.cfg` is based on variables related to separated sections:

* *global* in `haproxy_globals`
* *defaults* in `haproxy_defaults`
* *stats* in `haproxy_stats`
* *listens/frontends/backends* in `haproxy_binds`

Each configuration variable entry is transformed to `haproxy.cfg` setting, according to the following rules:

* **string** value to single config space separated entry

  ```yaml
  bind: "*.443"`
  ```

  becomes

  ```conf
  bind *.443
  ```

* **list** to multiple config entries with the same key

  ```yaml
  option:
    - "dontlognull"
    - "http-server-close"
    - "redispatch"
  ```

  becomes

  ```conf
  option dontlognull
  option http-server-close
  option redispatc
  ```

Special transform is applied to `haproxy_binds`.

* `servers` is list of backend servers (fqdns or ip addresses)
* `names` is list of backend servers names, replaced by `servers` if not defined</br>
   for consistency it should have the same number of items as `servers` however it is not mandatory
* `settings` is list of options appended to each backend server definition

```yaml
haproxy_binds:
  - listen: web
    backends:
      - servers:
          - server1.example.com
          - server2.example.com
        names:
          - backend1
        port: 80
        settings:
          - "check"
          - "inter 2s"
          - "rise 2"
          - "fall 3"
      - servers:
          - 192.168.0.10
        names:
          - backup
        port: 80
        settings:
          - "check"
          - "inter 2s"
          - "rise 2"
          - "fall 3"
          - "backup"
```

becomes

```conf
#---------------------------------------------------------------------
# web
#---------------------------------------------------------------------
listen web
  server backend1 server1.example.com:80 check inter 2s rise 2 fall 3
  server server2.example.com server2.example.com:80 check inter 2s rise 2 fall 3
  server backup 192.168.0.10:80 check inter 2s rise 2 fall 3 backup
```

Separate **frontends** and **backends** can also be defined.

```yaml
haproxy_binds:
  - frontend: "frontend-1"
    bind: "*:443"
    mode: "tcp"
    balance: "roundrobin"
    default_backend: "backend-0"

  - frontend: "frontend-2"
    bind: "*:8443"
    mode: "tcp"
    balance: "roundrobin"
    default_backend: "backend-0"

  - backend: "backend-0"
    backends:
      - servers:
          - server1.example.com
          - server2.example.com
        port: 443
        settings:
          - "check inter 1s"
```

becomes

```conf
#---------------------------------------------------------------------
# frontend-1
#---------------------------------------------------------------------
frontend frontend-1
  bind *:443
  mode tcp
  balance roundrobin
  default_backend backend-0

#---------------------------------------------------------------------
# frontend-2
#---------------------------------------------------------------------
frontend frontend-2
  bind *:8443
  mode tcp
  balance roundrobin
  default_backend backend-0

#---------------------------------------------------------------------
# backend-0
#---------------------------------------------------------------------
backend backend-0
  server server1.example.com server1.example.com:443 check inter 1s
  server server1.example.com server2.example.com:443 check inter 1s
```

Requirements
------------

* [ansible.builtin](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/index.html)
* [ansible.posix](https://docs.ansible.com/ansible/latest/collections/ansible/posix/index.html)
* [community.general](https://docs.ansible.com/ansible/latest/collections/community/general/)

Role Variables
--------------

* defaults

  * `main.yaml`

    ```yaml
    haproxy_name: ""        # name presented in HAproxy stats
    haproxy_firewalld: []   # firewalld settings
    ```

  * `globals.yaml` - **global** section in `haproxy.cfg`
  * `defaults.yaml` - **defaults** section in `haproxy.cfg`
  * `stats.yaml` - **stats** section in `haproxy.cfg`
  * `binds.yaml` - **listen** and/or **frontend/backend** sections in `haproxy.cfg`

    ```yaml
    haproxy_binds: []       # list of load balanced binds
      - listen: ""          # bind name, i.e. http
        bind: ""            # bind port, i.e. *:80
        mode: ""            # bind mode, i.e. http
        balance: ""         # bind balance, i.e. roundrobin, first
        backends: []        # list of backends definitions
          - servers: []     # list of backend servers
            names: []       # list of backend server names (see description)
            port: ""        # port of backend servers, i.e. 80
            settings: []    # list of backend servers options, i.e check

      - frontend: ""        # frontend name, i.e. https
        bind: ""            # bind port, i.e. *:443
        mode: ""            # bind mode, i.e. http
        balance: ""         # bind balance, i.e. roundrobin, first
        default_backend: "" # name of default backend
        backends: []        # list of backends definitions
          - servers: []     # list of backend servers
            names: []       # list of backend server names (see description)
            port: ""        # port of backend servers, i.e. 443
            settings: []    # list of backend servers options, i.e check
    ```

* vars

  ```yaml
  haproxy_pkgs: []            # list of HAproxy software packages
  haproxy_config: {}          # HAproxy config files attributes
  haproxy_service: ""         # systemd (or equivalent) service name
  ```

Dependencies
------------

* [service](https://github.com/mario-slowinski/service)
  * [software](https://github.com/mario-slowinski/software)

Tags
----

* haproxy.config
* haproxy.selinux
  * haproxy.selinux.sebool
  * haproxy.selinux.seport

Example Playbook
----------------

* `requirements.yml`

  ```yaml
  - name: haproxy
    src: https://github.com/mario-slowinski/haproxy
  ```

* playbook usage

  ```yaml
  - hosts: servers
    gather_facts: false
    roles:
      - role: haproxy
  ```

License
-------

[GPL-3.0](https://www.gnu.org/licenses/gpl-3.0.html)

Author Information
------------------

[mario.slowinski@gmail.com](mailto:mario.slowinski@gmail.com)
