# Odoo role

Deploy Odoo Docker container.

## Usage

Configure the role.

```yml
# https://hub.docker.com/r/mintsystem/odoo/
odoo_revision: "16.0.20240603"
odoo_image: mintsystem/odoo:16.0.20240603
odoo_build_image: true # default: false
odoo_build_dockerfile: | # default: ""
  RUN uv pip install prometheus-client
odoo_hostname: odoo01
odoo_config_map: # default: - name: prod
  - name: prod
  - name: int
odoo_timezone: Europe/Paris # default: Europe/Zurich
odoo_description: Odoo14 # default: Odoo
odoo_state: stopped # default: started
odoo_ports: # default: []
 - "127.0.0.1:8069:8069"
odoo_data_dir: /usr/share/odoo # default: "/usr/share/{{ odoo_hostname }}"
odoo_volume_name: odoo_data01 # default: "{{ odoo_hostname}}"
odoo_postgres_hostname: postgres01
odoo_postgres_user: odoo # default: {{ postgres_user }}"
odoo_postgres_password: # default: "{{ vault_postgres_password }}"
odoo_backup_set: # See restic_backup_set var in role restic
odoo_master_password: # default: "{{ vault_odoo_master_password }}"
odoo_dbfilter: ^%h$ # default: ^%d$
odoo_list_db: "true" # default: "false"
odoo_proxy_mode: "false" # default: "true"
odoo_python_install: prometheus_client
odoo_workers: 0 # default: 4
odoo_conf_limit_request: 4096 # default: 8192
odoo_conf_limit_time_cpu: 300 # default: 600
odoo_conf_limit_time_real: 600 # default: 1200
```

And include it in your playbook.

```yml
- hosts: odoo
  roles:
  - role: odoo
```

## Docs

### Nginx config

Setup this Nginx configuration for the `odoo01` host:

```yaml
nginx_http_options: |
  map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
  }
nginx_proxies:
  - src_hostname: odoo.example.com
    dest_hostname: odoo01
    dest_port: 8069
    exporter: odoo
    options: |
      include /etc/nginx/conf.d/proxy-params.conf;
      include /etc/nginx/conf.d/proxies/odoo-exporter.nginx;
    locations:
      - path: /websocket
        dest_hostname: odoo01
        dest_port: 8072
        options: |
          include /etc/nginx/conf.d/proxy-params.conf;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection $connection_upgrade;
```

### Calculate workers

The amount of workers to be set depends the amount CPUs available on the host.

Get the amount of CPUs on all hosts:

```bash
ansible -i inventories/setup all -m shell -a "grep -c ^processor /proc/cpuinfo"
```

Use rule of thumb: (#CPU * 2) + 1

In case of 4 CPUs it is 8 workers and 1 cron worker.

### Configure workers

If Odoo should make use of workers `workers = x>0` then this proxy config must be applied:

```yaml
nginx_proxies:
  - src_hostname: odoo.example.com
    dest_hostname: odoo01
    dest_port: 8069
    options: |
      location /longpolling {
        proxy_pass http://odoo01:8072;
        include /etc/nginx/conf.d/proxy-params.conf;
      }
```

Once workers are active the longpolling port moves from 8069 to 8072 and therefore a redirect is required.

### Official Odoo image

Configure these vars when using the official Odoo image:

```yml
# https://hub.docker.com/_/odoo/
odoo_revision: "16.0.20240603"
odoo_image: odoo@sha256:df0276cdb0ff8bb7883058071daf898d90fdbf13045ae96d131584660878da84
odoo_conf: | # default: ""
  server_wide_modules = base,web,dbfilter_from_header
```
