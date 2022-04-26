Role Name
=========
Filebeat role to help install, configure and uninstall Filebeat.

This role has been designed to allow for granular control over input and module declaration you can define modules/inputs at an all groups level, group_var level and host_var level 

Requirements
------------

No requirements for ansible but will require an elastic stack for Filebeat to report to

Role Tags
--------------

- install
- configure
- uninstall

Role Variables
--------------

```yaml

filebeat_fields_under_root: true # Set filebeat fields under root

filebeat_config: # Set input/module reload settings
  inputs:
    reload: true
    time: 30s
  modules:
    reload: true
    time: 30s

filebeat_output: # Set elasticsearch or logstash output
  name: elasticsearch
  hosts:
    - localhost
  port: 9200
  api_key:
  username:
  password:

filebeat_ssl:
  dir: /foo/bar
  certificate_authorities: foo.ca
  certificate_file: foo.crt
  key_file: foo.key
  insecure: true

filebeat_autodiscover: # Set's autodiscover, you can follow any configuration defined [here](https://www.elastic.co/guide/en/beats/filebeat/current/configuration-autodiscover.html) see the examples for more details

# set module/inputs at different levels (all are combined into a single dict one for modules one for input)
all_filebeat_modules:

group_filebeat_modules:

host_filebeat_modules:

all_filebeat_inputs:

group_filebeat_inputs:

host_filebeat_inputs:


```


Example Playbook
----------------

playbook.yml
```yaml
    - hosts: all
      vars
        filebeat_fields_under_root: true
        filebeat_config:
          inputs:
            reload: true
            time: 30s
          modules:
            reload: true
            time: 30s
      roles:
         - { role: chazragg.filebeat }
```
group_vars/all.yml
```yaml
filebeat_output:
  name: elasticsearch
  hosts:
    - localhost
  port: 9200

all_filebeat_modules:
  - module: system
    syslog:
      enabled: true
    auth:
      enabled: true

```

group_vars/docker.yml
```yaml

filebeat_autodiscover:
  autodiscover:
    providers:
      - type: docker
        ssl: true
        templates:
        - condition:
            contains:
              docker.container.image: nginx
          config:
            - module: nginx
              access:
                enabled: true
                input:
                  type: container
                  paths:
                    - "{{ docker_home }}/${data.docker.container.id}/*.log"
                  stream: stdout
              error:
                enabled: true
                input:
                  type: container
                  paths:
                    - "{{ docker_home }}/${data.docker.container.id}/*.log"
                  stream: stderr

```

group_vars/redis.yml
```yaml
group_filebeat_modules:
  - module: redis
    log:
      enabled: true
      var.paths: ["/path/to/log/redis/redis-server.log*"]
    slowlog:
      enabled: true
      var.hosts: ["localhost:6378"]
      var.password: "YOUR_PASSWORD"
```


host_vars/prdpython02.yml
```yaml
host_filebeat_inputs:
  - type: filestream
    paths:
      -/var/log/pythonApp/*.log
```

