Ansible Role: Flood-docker
==========================

Install Flood Docker Compose project.

- https://flood.js.org/
- https://github.com/jesec/flood

Requirements
------------

Requires the following to be installed:
- docker
- docker compose

Role Variables
--------------

Common Docker projects variables:

```yaml
# Base directory for Docker projects
docker_projects_path: # /var/apps
```

Available role variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
# Docker project variables

flood_project_name: flood

# Port targeted by Traefik router
flood_traefik_loadbalancer_server_port: "{{ flood_environment_options.port | default(3000) }}"

# Main service additional docker-compose options (ex: cpu_shares, deploy, ...)
flood_service_additional_options: |
  #ports:
  #  - 3000:3000

# Flood project variables

# jesec/flood image version
flood_version: latest

# Flood Docker environment variables
# https://github.com/jesec/flood/blob/master/config.ts
flood_environment_options:

  auth: default # Access control and user management method (default | none)

  # baseuri:                            # This URI will prefix all of Flood's HTTP requests
  # rundir:  /config/.local/share/flood # Where to store Flood's runtime files (eg. database)
  # host:    127.0.0.1                  # The host that Flood should listen for web connections on
  # port:    3000                       # The port that Flood should listen for web connections on

  # allowedpath: # Allowed path for file operations, can be called multiple times

  # secret:                   # A unique secret, a random one will be generated if not provided
  # disable-rate-limit: false # Disable api request limit except for login

  ## Deluge
  # dehost:   # Host of Deluge RPC interface
  # deport:   # Port of Deluge RPC interface
  # deuser:   # Username of Deluge RPC interface
  # depass:   # Password of Deluge RPC interface

  ## rTorrent
  # rthost:   # Host of rTorrent's SCGI interface
  # rtport:   # Port of rTorrent's SCGI interface
  # rtsocket: # Path to rTorrent's SCGI unix socket

  ## qBittorrent
  # qburl:    # URL to qBittorrent Web API
  # qbuser:   # Username of qBittorrent Web API
  # qbpass:   # Password of qBittorrent Web API

  ## Transmission
  # trurl:    # URL to Transmission RPC interface
  # truser:   # Username of Transmission RPC interface
  # trpass:   # Password of Transmission RPC interface

  ## SSL settings
  # ssl:     false # Enable SSL, key.pem and fullchain.pem needed in runtime directory
  # sslkey:        # Depends on ssl: Absolute path to private key for SSL
  # sslcert:       # Depends on ssl: Absolute path to fullchain cert for SSL

  ## Advanced settings
  # assets:           true           # ADVANCED: Serve static assets
  # dbclean:          1000 * 60 * 60 # ADVANCED: Interval between database purge
  # maxhistorystates: 30             # ADVANCED: Number of records of torrent download and upload speeds
  # clientpoll:       1000 * 2       # ADVANCED: How often (in ms) Flood will request the torrent list
  # clientpollidle:   1000 * 60 * 15 # ADVANCED: How often (in ms) Flood will request the torrent list when no user is present
  # rtorrent:         false          # ADVANCED: rTorrent daemon managed by Flood
  # rtconfig:                        # ADVANCED: rtorrent.rc for managed rTorrent daemon

# Volumes for filesystem operations
# ⚠️ Flood must have the same filesystem view as the torrent client
flood_volumes:
# Ex:
#  - /example/downloads:/downloads
#  - /example/watch:/watch
```

Dependencies
------------

This role depends on :
- [djuuu.docker_project](https://github.com/Djuuu/ansible-role-docker-project)

Some variables allow integration with:
- [djuuu.traefik_docker](https://github.com/Djuuu/ansible-role-traefik-docker)

Example Playbook
----------------

```yaml
- hosts: example
  gather_facts: false

  roles:
    - djuuu.flood_docker
```

### Integration in your BitTorrent client's Docker Compose project

The `template_service` task will register a `flood_service_yaml` variable that can be used in other projects.

Use the following variables to specify Flood's backend service dependency 
(Docker Compose [`depends_on`](https://docs.docker.com/reference/compose-file/services/#depends_on) attribute): 
- `flood_backend_service`:  
  key of your BitTorrent backend service in the main _docker-compose.yml_
- `flood_backend_condition`:  
  condition under which dependency is considered satisfied (defaults to `service_started`)

* `example_playbook.yml`
  ```yaml
  - hosts: example
    gather_facts: false
  
    pre_tasks:
      - name: Template Flood service yaml
        vars:
          flood_backend_service: example 
          flood_backend_condition: service_healthy # defaults to service_started
        ansible.builtin.include_role:
          name: djuuu.flood_docker
          tasks_from: template_service
  
    roles:
      - example_docker_role
  ```

* `example_docker_role/templates/docker-compose.yml.j2`
  ```yaml
  services:

    example:
      # ...

    {{ flood_service_yaml | default('') | indent(2) }}
  ```

License
-------

Beerware License
