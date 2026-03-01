# Taller de servidores Linux - 2026

El proyecto consiste en la creación de playbooks para la implementación de una infraestructura básica automatizada con Ansible.

Se trata de la instalación del servicio NFS en servidores CentOS, el montaje de estos directorios en Ubuntu y un servidor HTTP que expone los servicios del share en una web.

## Pre-requisitos

- Instalación de VM "Bastion" para administración de los servidores. 
- Instalación de 2 VM con CentOS
- Instalación de 2 VM con Ubuntu

Todas las VM requieren de 2 vCPU y 2Gb de RAM. Requerirán de un disco de 20Gb

### Particionamiento:

- /home - 2Gb
- / - 10Gb
- /Var - 3Gb
- /boot - 1Gb
- /swap - 4Gb

## Topología

Todos los servidores pertenecerán a la subred 192.168.10.0/24

- Bastion - 192.168.10.1
- Centos01 - 192.168.10.11
- Centos02 - 192.168.10.12
- Ubuntu01 - 192.168.10.21
- Ubuntu01 - 192.168.10.22