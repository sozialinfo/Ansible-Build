# Ansible Playbooks

Collection of Ansible playbooks and roles.

## Playbooks

* setup - Setup server
* odoo - Deploys Odoo instance
* wiki - Deploys Nextcloud, Fathom, Moodle and BookStack instance
* proxy - Deploys Nginx proxy instance
* monitor - Deploys ELK cluster
* backup - Deploys restic server and client

## Roles

* [docker](roles/docker/README.md) - Install Docker for Ubuntu
* [docker-network](roles/docker-network/README.md) - Define internal Docker network
* [postgres](roles/postgres/README.md) - Deploy PostgreSQL database container
* [openldap](roles/openldap/README.md) - Deploy OpenLDAP Docker container
* [odoo](roles/odoo/README.md) - Deploy Odoo container
* [debug](roles/debug/README.md) - Debug Ansible variables
* [modescurity](roles/modescurity/README.md) - Download and configure ModSecurity with OWASP CRS
* [nginx](roles/nginx/README.md) - Deploy Nginx proxy with Let's Encrypt certificates and ModSecurity
* [clean](roles/clean/README.md) - Cleanup Ansible roles
* [mysql](roles/mysql/README.md) - Deploy MySQL database container
* [bookstack](roles/bookstack/README.md) - Deploy BookStack Docker container
* [nextcloud](roles/nextcloud/README.md) - Deploy Nextcloud container
* [moodle](roles/moodle/README.md) - Deploy Moodle container
* [restic-server](roles/restic-server/README.md) - Deploy Restic server container
* [iam](roles/iam/README.md) - Configures users and groups
* [restic-client](roles/restic-client/README.md) - Configure Restic client backup jobs
* [elasticsearch](roles/elasticsearch/README.md) - Deploy ElasticSearch Docker cluster
* [kibana](roles/elasticsearch/README.md) - Deploy Kibana Docker container

## Usage

Clone this repository.

`git clone https://github.com/Mint-System/Ansible-Playbooks.git && cd Ansible-Playbooks`

### Setup

Set a password to encry the Ansible vault.

`export VAULTPASSWORD=PASSWORD`

Create a password file.

`echo "$VAULTPASSWORD" > .vault_pass`

Make it executable.

`chmod 600 .vault_pass`

Install jmespath with pip.

`pip3 install jmespath`

Create an inventory and configure a role.

### Deployment

List inventory

`ansible-inventory --list -y -i inventories/odoo`

Test connection

`ansible all -m ping -i inventories/odoo`

Deploy odoo stack

`ansible-playbook -i inventories/odoo odoo.yml`

Deploy role only

`ansible-playbook -i inventories/odoo odoo.yml -t docker`

Deploy role to specific host

`ansible-playbook -i inventories/odoo odoo.yml -t docker -l host.example.com`

Deploy role to specific group with non-default user

`ansible-playbook -i inventories/odoo docker.yml -t docker -l europe -u username`

Clean odoo stack

`ansible-playbook -i inventories/odoo clean.yml -t odoo,odoo-volume`

Clean role only

`ansible-playbook -i inventories/odoo clean.yml -t docker`

## Development

### Quality

Lint the project using Ansible lint.

`ansible-lint odoo.yml clean.yml`

### Config

Whenever possible use env variables to configure the container.

**Env Config**

```yml
    env:
      POSTGRES_USER: "{{ postgres_user }}"
      POSTGRES_PASSWORD: "{{ postgres_password }}"
      POSTGRES_DB: "{{ postgres_db }}"
```

### Data

To persist data use Docker volumes.

**Volume Mount**

Mount the folder without subfolder.

```yml
    volumes:
      - "{{ postgres_volume_name }}:/var/lib/postgresql/data"
```

For Ansible config files use file mounts.

**Bind Mount**

```yml
    volumes:
      - "{{ nginx_data_dir }}/:/etc/nginx/conf.d/:ro"
```

### Docs

Every role folder must contain a `README.md` file.
