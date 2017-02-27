# ansible-pimatic

An Ansible role to install and configure [pimatic](https://pimatic.org) - a smart home automation software for the
[Raspberry Pi](https://www.raspberrypi.org/).

## Requirements

* Raspberry Pi with Raspbian OS
* network access to the Raspberry Pi and sshd enabled
* a user with sudo permissions

## Role Variables

### mandatory

* **pimatic_nodejs_arch:** Has to be set. Choose node.js hardware architecture, typically **armv6l** (Pi Model A, B, B+ or Zero) 
or **armv7l** (Pi 2 Model B or Pi 3 Model B).

### optional

* **pimatic_app_dir**
  * default: "pimatic-app"
  * description: Name of the directory in the user's home directory where pimatic should be installed to. E.g. with user `pi` the final
  installation directory will become `/home/pi/pimatic-app`.

* **pimatic_port**
  * default: 8080
  * description: Port pimatic's web frontend will listen on. We will also wait_for pimatic to start up on that port. You may want to set it
  to **443** when you enable **pimatic_ssl**.

* **pimatic_config_template**
  * default: "config.json.j2"
  * description: The role comes with pimatic's standard configuration. Pimatic's httpServer will be started on **pimatic_port**.
  Usually you want to provide your own pimatic configuration template. E.g. set **pimatic_config_template** to **pimatic_config.json.j2**
  and create a file of that name in the `templates` directory relative to your ansible playbook.

* **pimatic_waitfor_startup** 
  * default: true
  * description: Configure whether the role wait for pimatic to start up. Depending on the hardware we are running on and depending on the
  configured plugins this may take up to 1h.

* **pimatic_ssl** 
  * default: false
  * description: Configure whether we should generate self signed ssl certificates. If set to **true** the role will provide the variables
  **pimatic_ssl_privkey** and **pimatic_ssl_cert** with the paths to the generated ssl private key and certificate.


### provided

* **pimatic_ssl_privkey:** Provided if **pimatic_ssl** is set to **true**. You may want to use this variable in your 
**pimatic_config_template** to setup a `httpsServer`.

* **pimatic_ssl_cert:** Provided if **pimatic_ssl** is set to **true**. You may want to use this variable in your 
**pimatic_config_template** to setup a `httpsServer`.

## Example Playbook

Typical playbook run:
```sh
$ ansible-playbook -i inventory pimatic.yml
```

*inventory*
```INI
[raspberrypi]
pizero ansible_host=192.168.0.101 ansible_user=pi ansible_ssh_pass=raspberry 
```

*pimatic.yml*
```YAML
- hosts: raspberrypi
  
  roles:
    - pimatic
  
  vars:
    pimatic_nodejs_arch: "armv6l"
    pimatic_port: 443
    pimatic_config_template: "pimatic_config.json.j2"
    pimatic_ssl: true
    pimatic_admin_password: "verysecret"
```

*templates/pimatic_config.json.j2*
```jinja2
{
  "settings": {
    "httpServer": {
      "enabled": false
    },
    "httpsServer": {
      "enabled": true,
      "port": {{ pimatic_port }},
      "hostname": "",
      "keyFile": "{{ pimatic_ssl_privkey }}",
      "certFile": "{{ pimatic_ssl_cert }}"
    },
    "database": {
    }
  },
  "plugins": [
    {
      "plugin": "cron"
    },
    {
      "plugin": "mobile-frontend"
    }
  ],
  "devices": [

  ],
  "rules": [

  ],
  "pages": [
    {
      "id": "favourite",
      "name": "Favourites",
      "devices": []
    }
  ],
  "groups": [

  ],
  "users": [
    {
      "username": "admin",
      "password": "{{ pimatic_admin_password }}",
      "role": "admin"
    }
  ],
  "roles": [
    {
      "name": "admin",
      "permissions": {
        "pages": "write",
        "rules": "write",
        "variables": "write",
        "messages": "write",
        "events": "write",
        "devices": "write",
        "groups": "write",
        "plugins": "write",
        "updates": "write",
        "database": "write",
        "config": "write",
        "controlDevices": true,
        "restart": true
      }
    }
  ]
}
```

## Install the role via Ansible Galaxy

Typical run:
```sh
$ ansible-galaxy install -r requirements.yml
```
*requirements.yml*
```YAML
- name: pimatic
  src: https://github.com/layereight/ansible-pimatic
  version: "1.2"
```
* also see the [Ansible Galaxy documentation](http://docs.ansible.com/ansible/galaxy.html)
