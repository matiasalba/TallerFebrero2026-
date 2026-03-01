# Taller de servidores Linux - 2026

El proyecto consiste en la creación de playbooks para la implementación de una infraestructura básica automatizada con Ansible.

Se trata de la instalación del servicio NFS en servidores CentOS, el montaje de estos directorios en Ubuntu y un servidor HTTP que expone los servicios del share en una web.

## Pre-requisitos

- Instalación de VM "Bastion" para administración de los servidores. **Debe tener Ansible instalado**
- Instalación de 2 VM con CentOS
- Instalación de 2 VM con Ubuntu

Todas las VM requieren de 2 vCPU y 2Gb de RAM. Requerirán de un disco de 20Gb

### Particionamiento:

- **/home** - 2Gb
- **/**- 10Gb
- **/Var** - 3Gb
- **/boot** - 1Gb
- **/swap** - 4Gb

## Topología

Todos los servidores pertenecerán a la subred 192.168.10.0/24

- **Bastion** - 192.168.10.1
- **Centos01** - 192.168.10.11
- **Centos02** - 192.168.10.12
- **Ubuntu01** - 192.168.10.21
- **Ubuntu02** - 192.168.10.22

## Ejecución

El proyecto consiste en 5 playbooks. 

- **hardening.yaml** - Se encarga de actualizar y aplicar medidas básicas de seguridad
- **nfs-client.yaml** - Configura y monta los share en los clientes Ubuntu 
- **nfs-server-yaml** - configura e inicia servicios en el servidor CentOS01
- **ubuntu-ufw** - habilita los puertos en el firewall de Ubuntu para habilitar tráfico ssh, tráfico saliente y tráfico entrante. 
- **webserver.yaml** - Configura servidor web en los servidores Ubuntu. 

Adicionalmente, hay un playbook adicional, que corre todos los anteriormente mencionados. Se encuentra en la carpeta playbook y se llama **"site.yaml"**

### Ejecución de playbook: 

```ansible
ansible-playbook -i inventories/hosts.ini playbooks/sites.yaml --ask-become-pass
```

## Verificación de funcionamiento. 

Para verificar que los playbooks correrán correctamente: 

```ansible
ansible-playbook -i inventories/hosts.ini playbooks/sites.yaml --check --ask-become-pass
```
Respuesta esperada en caso de haber sido corrido en el pasado:

```bash
[WARNING]: Collection ansible.posix does not support Ansible version 2.14.18
[WARNING]: Collection community.general does not support Ansible version 2.14.18

PLAY [linux] ***************************************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************
ok: [centos02]
ok: [ubuntu02]
ok: [centos01]
ok: [ubuntu01]

TASK [Update all installed packages] ***************************************************************************************************************************************
skipping: [ubuntu01]
skipping: [ubuntu02]
ok: [centos02]
ok: [centos01]

TASK [Update all installed packages in Ubuntu] *****************************************************************************************************************************
skipping: [centos01]
skipping: [centos02]
changed: [ubuntu02]
changed: [ubuntu01]

TASK [Disable root login Centos] *******************************************************************************************************************************************
skipping: [ubuntu01]
skipping: [ubuntu02]
ok: [centos02]
ok: [centos01]

TASK [Disable root login Ubuntu] *******************************************************************************************************************************************
skipping: [centos01]
skipping: [centos02]
ok: [ubuntu02]
ok: [ubuntu01]

RUNNING HANDLER [Reboot all servers] ***************************************************************************************************************************************
changed: [ubuntu01]
changed: [ubuntu02]

PLAY [fileserver] **********************************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************
ok: [nfs01]

TASK [Install nfs server] **************************************************************************************************************************************************
ok: [nfs01]

TASK [Create shared directory] *********************************************************************************************************************************************
ok: [nfs01]

TASK [Export shared directory] *********************************************************************************************************************************************
ok: [nfs01]

TASK [Service NFS is started] **********************************************************************************************************************************************
ok: [nfs01]

TASK [NFS enabled in firewalld] ********************************************************************************************************************************************
ok: [nfs01]

TASK [Copy test file to server] ********************************************************************************************************************************************
changed: [nfs01]

PLAY [ubuntu] **************************************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************
ok: [ubuntu02]
ok: [ubuntu01]

TASK [Install required packages for automount] *****************************************************************************************************************************
ok: [ubuntu02] => (item=autofs)
ok: [ubuntu01] => (item=autofs)
ok: [ubuntu02] => (item=nfs-common)
ok: [ubuntu01] => (item=nfs-common)
ok: [ubuntu02] => (item=python3)
ok: [ubuntu01] => (item=python3)

TASK [Add nfs.autofs to auto.master.d] *************************************************************************************************************************************
ok: [ubuntu02]
ok: [ubuntu01]

TASK [Add nfs mount map file] **********************************************************************************************************************************************
ok: [ubuntu01]
ok: [ubuntu02]

TASK [Start autofs] ********************************************************************************************************************************************************
ok: [ubuntu01]
ok: [ubuntu02]

PLAY [ubuntu] **************************************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************
ok: [ubuntu02]
ok: [ubuntu01]

TASK [Reset ufw] ***********************************************************************************************************************************************************
changed: [ubuntu01]
changed: [ubuntu02]

TASK [Allow outgoing traffic] **********************************************************************************************************************************************
ok: [ubuntu01]
ok: [ubuntu02]

TASK [Deny incomming traffic] **********************************************************************************************************************************************
ok: [ubuntu01]
ok: [ubuntu02]

TASK [Allow SSH] ***********************************************************************************************************************************************************
ok: [ubuntu02]
ok: [ubuntu01]

TASK [Enable ufw firewall] *************************************************************************************************************************************************
ok: [ubuntu01]
ok: [ubuntu02]

PLAY [ubuntu] **************************************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************
ok: [ubuntu02]
ok: [ubuntu01]

TASK [Copy webserver unit file] ********************************************************************************************************************************************
ok: [ubuntu02]
ok: [ubuntu01]

TASK [Reload systemd daemon] ***********************************************************************************************************************************************
ok: [ubuntu01]
ok: [ubuntu02]

TASK [Start webserver] *****************************************************************************************************************************************************
ok: [ubuntu01]
ok: [ubuntu02]

TASK [Open port 8080] ******************************************************************************************************************************************************
ok: [ubuntu01]
ok: [ubuntu02]

PLAY RECAP *****************************************************************************************************************************************************************
centos01                   : ok=3    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
centos02                   : ok=3    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
nfs01                      : ok=7    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu01                   : ok=20   changed=3    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
ubuntu02                   : ok=20   changed=3    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0  
``

Para verificar que el webserver responde: 

```bash
ansible -i inventories/hosts.ini ubuntu -m command -a 'curl http://localhost:8080'
```

Respuesta esperada:
```bash
<!DOCTYPE HTML>
<html lang="en">
...
<li><a href="README-NFS.txt">README-NFS.txt</a></li>
...
```

