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

On mets bien le chemin de notre clé ssh pour pouvoir se connecter.  
On place également l'adresse de notre machine distante.  
Attention à l'indentation.

On test avec la commande de ping:
```bash
arthur@DESKTOP-CU3J2TG:~/dev/DevOps$ ansible all -i ansible/inventories/setup.yml -m ping
arthur.saunier.takima.cloud | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
```

### Facts

Cmd pour récupérer l'OS de notre serveur:
> ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"

Reponse:
```bash
arthur@DESKTOP-CU3J2TG:~/dev/DevOps/ansible$ ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"
arthur.saunier.takima.cloud | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution": "CentOS",
        "ansible_distribution_file_parsed": true,
        "ansible_distribution_file_path": "/etc/centos-release",
        "ansible_distribution_file_variety": "CentOS",
        "ansible_distribution_major_version": "8",
        "ansible_distribution_release": "Stream",
        "ansible_distribution_version": "8",
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false
}
```

Delete Httpd:
```bash
arthur@DESKTOP-CU3J2TG:~/dev/DevOps/ansible$ ansible all -i inventories/setup.yml -m yum -a "name=httpd state=absent" --become
arthur.saunier.takima.cloud | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": true,
    "msg": "",
    "rc": 0,
    "results": [
        "Removed: mod_http2-1.15.7-3.module_el8.4.0+778+c970deab.x86_64",
        "Removed: httpd-2.4.37-43.module_el8.5.0+1022+b541f3b1.x86_64"
    ]
}
```

## Playbooks

### First playbook

Créer le playbook en racine de /ansible

#### playbook
```yml
- hosts: all
  gather_facts: false
  become: yes

  tasks:
    - name: Test connection
      ping:
```

On test le playbook:
> ansible-playbook -i inventories/setup.yml playbook.yml

Si on souhaite vérifier la syntax du playbook: 
> --syntax-check

```bash
arthur@DESKTOP-CU3J2TG:~/dev/DevOps/ansible$ ansible-playbook -i inventories/setup.yml playbook.yml

PLAY [all] **************************************************************************************************************************************************************************

TASK [Test connection] **************************************************************************************************************************************************************
ok: [arthur.saunier.takima.cloud]

PLAY RECAP **************************************************************************************************************************************************************************
arthur.saunier.takima.cloud : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### Advanced playbook
