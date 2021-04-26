haproxy
=======

Ansible role to manage HAproxy.

Each load balanced application configuration is stored in separate file in `conf.d` directory and then assembled into single `haproxy.cfg` file which is the only file read by HAproxy.

Requirements
------------

* [ansible.builtin](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/index.html)
* [ansible.posix](https://docs.ansible.com/ansible/latest/collections/ansible/posix/index.html)
* [community.general](https://docs.ansible.com/ansible/latest/collections/community/general/)

Role Variables
--------------

* defaults

  ```yaml
  haproxy_firewalld: []       # firewalld settings

  haproxy_applications: []    # list of load balanced applications
    - name: ""                # name of load balanced application
      bind: {}                # bind to local interface and frontend port
      servers: []             # list of backend servers, ansible inventory group
      port:                   # backend port number
      settings: []            # list of HAproxy options
  ```

* vars

  ```yaml
  haproxy_pkgs: []            # list of HAproxy software packages
  haproxy_config: {}          # HAproxy config files attributes
  ```

Dependencies
------------

* [service](https://github.com/mario-slowinski/service)
  * [software](https://github.com/mario-slowinski/software)

Tags
----

* haproxy.config
  * haproxy.config.directory
  * haproxy.config.base
  * haproxy.config.include
  * haproxy.config.assemble
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
