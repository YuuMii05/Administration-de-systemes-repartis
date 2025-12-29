# Administration-de-systemes-repartis

**VERSION FRANCAIS**

Ce projet automatise le déploiement d’une infrastructure hétérogène via Vagrant et Ansible (Infrastructure as Code). Il comprend une machine Ubuntu de contrôle, deux nœuds RedHat en cluster HA (Pacemaker/Corosync) avec Nginx/Samba derrière une VIP flottante, et un Windows Server piloté en WinRM (AD + hardening), avec les preuves et vérifications décrites dans le README

![shema darchitecture](https://github.com/user-attachments/assets/730de916-854b-4390-b372-f052ce1988b1)

**Mission 1 (Démarrage du lab)**

**Objectif**
Démarrer le lab multi‑VM avec Vagrant/VMware et vérifier que toutes les machines sont actives et joignables depuis le nœud de contrôle (**admin**). Vagrant sert à définir et gérer un environnement reproductible à partir d’un Vagrantfile.[1]

**Environnement**
Machines créées par Vagrant (provider VMware) :
- admin (nœud de contrôle Linux)
- node01 (cible Linux)
- node02 (cible Linux)
- winsrv (cible Windows)
<img width="997" height="592" alt="screenshotsvagrant-up png" src="https://github.com/user-attachments/assets/e49ab84e-7410-4711-b805-25f4fef93ce6" />

**Étapes réalisées**

1) Démarrer et vérifier les VMs
Depuis l’hôte (dossier du projet) :

vagrant up
vagrant status
Résultat : toutes les VMs sont en état running (vmware_desktop).
<img width="1105" height="823" alt="screenshotsvagrant-status png" src="https://github.com/user-attachments/assets/f1c365ec-6b0e-4193-b665-2b23ad6c8bdc" />

2) Se connecter au nœud admin (contrôle)
Depuis l’hôte :
powershell
vagrant ssh admin
<img width="1462" height="775" alt="screenshotsvagrant-ssh-admin png (2)" src="https://github.com/user-attachments/assets/c3850798-8b11-46fb-ab09-d2f1edc1aadd" />

3) Vérifier la configuration réseau de admin
Sur admin :
bash
ip -4 addr
ip route
Résultat : admin possède :
une interface “NAT” pour l’accès Internet (ex : 192.168.80.x)
une interface “réseau privé” pour la communication inter‑VM (ex : 192.168.77.10/24)
<img width="1047" height="820" alt="screenshotsadmin-ip-addr png" src="https://github.com/user-attachments/assets/5780f025-a0e1-4b8c-9b10-0afaddf0f33b" />

4) Vérifier la connectivité vers les nœuds Linux
Sur admin :
bash
ping -c 2 192.168.77.11   # node01
ping -c 2 192.168.77.12   # node02


**Mission 2 (Cluster HA + vérifications)**

**Objectif**
Configurer et valider la mise en place HA Linux (Pacemaker/Corosync) et vérifier la configuration firewall/SSH ainsi que la disponibilité des services depuis le nœud de contrôle (admin) à l’aide de commandes Ansible “ad‑hoc”. Un inventaire Ansible permet de regrouper des hôtes et d’exécuter des commandes sur un groupe.​

**Étapes réalisées**
1) Vérifier les services autorisés par le firewall (firewalld)
Depuis admin :
bash
ansible linux -i /vagrant/ansible/hosts.ini -b -m shell -a "firewall-cmd --list-services"
Constat : les services incluent high-availability, http, ssh, zabbix-agent.
<img width="928" height="513" alt="rendu 5" src="https://github.com/user-attachments/assets/96d7bccd-0f50-48c4-8cc7-196c465ea8fb" />

2) Vérifier services + ports (firewalld)
Depuis admin :
bash
ansible linux -i /vagrant/ansible/hosts.ini -b -m shell -a "firewall-cmd --list-all | egrep 'services:|ports:'"
<img width="947" height="747" alt="rendu 6 mission2" src="https://github.com/user-attachments/assets/f6522cb3-1620-46aa-bfb0-682cee5144ec" />

3) Vérifier le durcissement SSH (PermitRootLogin)
Depuis admin :
bash
ansible linux -i /vagrant/ansible/hosts.ini -b -m shell -a "sshd -T | grep -i permitrootlogin"
Constat : permitrootlogin no.
<img width="930" height="422" alt="rendu 7 mission2" src="https://github.com/user-attachments/assets/405f2eba-8317-4237-9771-3d2f1b2634a2" />

4) Vérifier l’état du cluster (Pacemaker)
Depuis admin :
bash
ansible node01 -i /vagrant/ansible/hosts.ini -b -m shell -a "pcs status"
<img width="942" height="425" alt="rendu 8 mission2" src="https://github.com/user-attachments/assets/5352b162-5341-4f83-abd0-cfd880041d31" />

5) Vérifier les contraintes (constraints)
Depuis admin :
bash
ansible node01 -i /vagrant/ansible/hosts.ini -b -m shell -a "pcs constraint"
<img width="1002" height="596" alt="rendu 4 1 group + contraint" src="https://github.com/user-attachments/assets/73da1dbc-8419-46d2-b7d2-38f2e74a5c25" />

<img width="1098" height="822" alt="rendu 4 2 group + contraint" src="https://github.com/user-attachments/assets/7df83b1f-9690-464f-95ac-b38281d61e0a" />

<img width="1830" height="232" alt="rendu 4 3 group + contraint" src="https://github.com/user-attachments/assets/eea9b913-428e-4aa6-9db0-e13ba6b9b34c" />

6) Vérifier la configuration des ressources
Depuis admin :
bash
ansible node01 -i /vagrant/ansible/hosts.ini -b -m shell -a "pcs resource config"

7) Valider l’accès HTTP via la VIP (nginx)
Depuis admin :
bash
ansible node01 -i /vagrant/ansible/hosts.ini -b -m shell -a "curl -I --max-time 2 http://192.168.77.100/"
Constat : réponse HTTP/1.1 200 OK et serveur nginx.
<img width="942" height="425" alt="rendu 8 mission2" src="https://github.com/user-attachments/assets/b39bfe8b-b07c-4a22-9dad-82d743eaafd0" />



**ENGLISH VERSION**

This project automates the deployment of a small heterogeneous infrastructure using Vagrant and Ansible (Infrastructure as Code). It includes an Ubuntu control node, two RedHat nodes in a Pacemaker/Corosync HA cluster hosting Nginx and Samba behind a floating VIP, and a Windows Server managed via WinRM for AD/hardening, with security and validation steps documented in the README.

**Mission 1 (Lab bootstrap)**

**Goal**
Bring up the multi‑VM lab with Vagrant/VMware and verify that all machines are running and reachable from the control node (admin). Vagrant is used to define and manage multi-VM environments from a single Vagrantfile.
​
**Environment**
Machines created by Vagrant (VMware provider):
admin (Linux control node)
node01 (Linux target)
node02 (Linux target)
winsrv (Windows target)
<img width="997" height="592" alt="screenshotsvagrant-up png" src="https://github.com/user-attachments/assets/e49ab84e-7410-4711-b805-25f4fef93ce6" />

**Steps performed**
1) Start and verify VMs
From the host (project folder):
powershell
vagrant up
vagrant status
Result: all VMs were in running (vmware_desktop).
<img width="1105" height="823" alt="screenshotsvagrant-status png" src="https://github.com/user-attachments/assets/f1c365ec-6b0e-4193-b665-2b23ad6c8bdc" />

3) Connect to admin (control node)
From the host:
powershell
vagrant ssh admin
<img width="1462" height="775" alt="screenshotsvagrant-ssh-admin png (2)" src="https://github.com/user-attachments/assets/c3850798-8b11-46fb-ab09-d2f1edc1aadd" />

3) Validate admin network configuration
On admin:
bash
ip -4 addr
ip route
Result: admin had:
a “NAT” interface for internet access (example: 192.168.80.x)
a private lab interface for inter-VM communication (example: 192.168.77.10/24)
<img width="1047" height="820" alt="screenshotsadmin-ip-addr png" src="https://github.com/user-attachments/assets/5780f025-a0e1-4b8c-9b10-0afaddf0f33b" />

4) Validate connectivity to Linux nodes
On admin:
bash
ping -c 2 192.168.77.11   # node01
ping -c 2 192.168.77.12   # node02


**Mission 2 (HA cluster + checks)**

**Goal**
Configure and validate the Linux HA setup (Pacemaker/Corosync) and verify firewall/SSH configuration and service availability from the control node (admin) using Ansible ad‑hoc commands. Ansible inventories and ad‑hoc commands are a standard way to run remote checks.

**Steps performed**
1) Verify firewall services on nodes
From admin:
bash
ansible linux -i /vagrant/ansible/hosts.ini -b -m shell -a "firewall-cmd --list-services"
Expected/observed: services include high-availability, http, ssh, zabbix-agent.
<img width="928" height="513" alt="rendu 5" src="https://github.com/user-attachments/assets/96d7bccd-0f50-48c4-8cc7-196c465ea8fb" />

2) Verify firewall services + ports
From admin:
bash
ansible linux -i /vagrant/ansible/hosts.ini -b -m shell -a "firewall-cmd --list-all | egrep 'services:|ports:'"
<img width="947" height="747" alt="rendu 6 mission2" src="https://github.com/user-attachments/assets/f6522cb3-1620-46aa-bfb0-682cee5144ec" />

3) Verify SSH hardening (PermitRootLogin)
From admin:
bash
ansible linux -i /vagrant/ansible/hosts.ini -b -m shell -a "sshd -T | grep -i permitrootlogin"
Expected/observed: permitrootlogin no.
<img width="930" height="422" alt="rendu 7 mission2" src="https://github.com/user-attachments/assets/405f2eba-8317-4237-9771-3d2f1b2634a2" />

4) Check cluster status (Pacemaker)
From admin:
bash
ansible node01 -i /vagrant/ansible/hosts.ini -b -m shell -a "pcs status"
<img width="942" height="425" alt="rendu 8 mission2" src="https://github.com/user-attachments/assets/5352b162-5341-4f83-abd0-cfd880041d31" />

5) Check constraints
From admin:
bash
ansible node01 -i /vagrant/ansible/hosts.ini -b -m shell -a "pcs constraint"
<img width="1002" height="596" alt="rendu 4 1 group + contraint" src="https://github.com/user-attachments/assets/73da1dbc-8419-46d2-b7d2-38f2e74a5c25" />

<img width="1098" height="822" alt="rendu 4 2 group + contraint" src="https://github.com/user-attachments/assets/7df83b1f-9690-464f-95ac-b38281d61e0a" />

<img width="1830" height="232" alt="rendu 4 3 group + contraint" src="https://github.com/user-attachments/assets/eea9b913-428e-4aa6-9db0-e13ba6b9b34c" />

6) Check resource configuration
From admin:
bash
ansible node01 -i /vagrant/ansible/hosts.ini -b -m shell -a "pcs resource config"

7) Validate VIP HTTP (nginx)
From admin:
bash
ansible node01 -i /vagrant/ansible/hosts.ini -b -m shell -a "curl -I --max-time 2 http://192.168.77.100/"
Expected/observed: HTTP/1.1 200 OK and server nginx.
<img width="942" height="425" alt="rendu 8 mission2" src="https://github.com/user-attachments/assets/b39bfe8b-b07c-4a22-9dad-82d743eaafd0" />

