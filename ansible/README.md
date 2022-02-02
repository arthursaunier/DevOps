# TP03

#### ssh cmd:
>ssh -i key/id_rsa centos@arthur.saunier.takima.cloud

## Intro

### inventories

#### setup.yml
```yml
all:
  vars:
    ansible_user: centos
    ansible_ssh_private_key_file: ../../../key/id_rsa
  children:
    prod:
    hosts: arthur.saunier.takima.cloud
```