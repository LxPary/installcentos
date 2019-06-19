# Installation
## Instale el clúster de OpenShift origen 3.11 en una única máquina ejecutando CentOS 7
### Los requisitos mínimos para openshift origin (OKD) 3.11 son al menos 16 GB de memoria, pero como mi máquina no tiene tanta capacidad, solo uso 8 GB de memoria y excluyo todas las comprobaciones de hardware en mi archivo de inventario. 
### Para que la instalación de openshift funcione sin problemas, necesita un servidor DNS apropiado e independiente. Consulte [aqui](DNS.md), sobre cómo configurar un servidor DNS muy fácil. El DNS se puede instalar en otra máquina virtual con probablemente 512 MB de memoria. 
1. Configuracion basica 

```
 DOMAIN=${DOMAIN:="domain.xxx"}
 IP=${IP:="192.168.1.xxx"}
 HOST_NAME=${HOST_NAME:="console"}
```

2. Cambiar Dominio local

```
$ hostnamectl set-hostname console
```

3. Agregar Dominio al Host

```
$ cat <<EOD > /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4  
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
${IP}       console.${DOMAIN} ${HOST_NAME} 
EOD
```

4. Ya que vamos a utilizar ssh ansible, sin contraseña, aunque solo sea una máquina

```
$ ssh-keygen
$ ssh-copy-id ${HOST_NAME}
```

5. Actualice el sistema operativo e instale los paquetes base

```
$ yum update -y; yum install wget git net-tools bind-utils yum-utils iptables-services bridge-utils bash-completion kexec-tools sos psacct -y; reboot
```

6. Instale el repositorio de epel y deshabilite el repositorio por defecto

```
$ yum install -y epel-release; 
$ sed -i -e 's/enabled=1/enabled=0/' /etc/yum/repos.d/epel.repo
```

7. Instale ansible y pyOpenSSL

```
$ yum install -y --enablerepo=epel ansible pyOpenSSL
```

8. Install docker

```
$ yum install -y docker-1.13.1
```

9. Clone el repositorio openshift-origin en github. Este repositorio proporcionará los libros de reproducción y los archivos de configuración necesarios

```
$ cd
$ git clone https://github.com/openshift/openshift-ansible
$ cd openshift-ansible
$ git checkout release-3.11
```

10. Genera una contraseña hash para tu primer usuario 

```
$ openssl passwd -apr1 poncualquiercontraseña
```

11. Prepare su archivo de inventario. Puede consultar en [aqui](https://docs.okd.io/3.11/install/configuring_inventory_file.html) el significado de cada una de las opciones en el siguiente archivo de inventario. Asegúrese de que todos los nombres de host utilizados en este archivo puedan resolverse por DNS 

```
$ cd
$ cat > ~/openshift-ansible/inventory.ini <<EOF
[OSEv3:children]
masters
nodes
etcd

[OSEv3:vars]
ansible_ssh_user=root
openshift_deployment_type=origin
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider'}]
#openshift_master_htpasswd_users={'admin': '$apr1$qpJB3Cls$PN7/HlUNqBXikBl.jnrHF.'}

openshift_public_hostname=console.${DOMAIN}
openshift_master_default_subdomain=apps.console.${DOMAIN}
openshift_disable_check=disk_availability,docker_storage,memory_availability,docker_image_availability

[masters]
${HOST_NAME}.${DOMAIN} openshift_schedulable=true 

[etcd]
${HOST_NAME}.${DOMAIN}

[nodes]
${HOST_NAME}.${DOMAIN} openshift_schedulable=true openshift_node_group_name="node-config-all-in-one"
EOF
```
12. Ejecute el libro de requisitos previos.yml. Este libro de jugadas se instalará requerido para la instalación openshift 
```
$ cd ~/openshift-ansible
$ ansible-playbook -i inventory.ini playbooks/prerequisites.yml
```

13. Ejecute el libro de jugadas de implementación para implementar su grupo de openshift
```
$ ansible-playbook -i inventory.ini playbooks/deploy_cluster.yml
```
14. Una vez que se haya completado la instalación, verifique su instalación revisando los nodos

```
$ oc get nodes
```

15. En el nodo maestro, agregue un nuevo usuario con HTPasswd.

```
$ htpasswd /etc/origin/master/htpasswd developer
```




[...](https://www.linuxwave.info/2019/05/install-openshift-origin-311-cluster-on.html)