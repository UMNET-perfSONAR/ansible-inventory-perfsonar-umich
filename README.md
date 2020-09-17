# ansible-umich-perfSONAR-core

ansible-umich-perfSONAR-core

Required:
  - account on its-perfsonar-bastion.dsc.umich.edu
  - account and ssh keys on all dells & lsi machines
  - account on all miservers
  - know vault password

How to use this repository:
1.  log on to its-perfsonar-bastion.dsc.umich.edu

```
ssh its-perfsonar-bastion.dsc.umich.edu
```

2.  Get the official perfSONAR Ansible playbook and bootstrap it

```
git clone https://github.com/perfsonar/ansible-playbook-perfsonar.git
cd ansible-playbook-perfsonar
ansible-galaxy install -r  requirements.yml --ignore-errors
./defaults.sh
```

3.  add this repository to the perfSONAR Ansible playbook

```
git clone git@gitlab.umich.edu:epcjr/ansible-umich-perfsonar-core.git
```

4.  Use Ansible ping to verify connectivity to targets.  ssh and become passwords are Kerberos.

```
ansible \
  --ask-pass \
  --ask-become-pass  \
  --ask-vault-pass \
  -i ansible-umich-perfsonar-core/inventory \
  all -m ping
```

5. Provision perfSONAR infrastructure

```
ansible-playbook \
  -i ansible-umich-perfsonar-core/inventory \
  --ask-pass \
  --ask-become-pass  \
  --ask-vault-pass \
  perfsonar.yml
```

```
ansible-playbook \
  -i ansible-umich-perfsonar-core/inventory \
  --ask-pass \
  --ask-become-pass  \
  --ask-vault-pass \
  --limit 'ps-psconfig-publishers' \
  perfsonar.yml
```

Provision AWS infrastructure

```
ansible-playbook \
  -i ansible-umich-perfsonar-core/inventory \
  --ask-pass \
  --ask-become-pass  \
  --ask-vault-pass \
  --limit 'ps-testpoints-aws' \
  perfsonar.yml
```

6. Provision local changes to perfSONAR infrastructure

```
ansible-playbook \
  -i ansible-umich-perfsonar-core/inventory \
  --ask-vault-pass \
  --limit 'nanopis' \
  ansible-umich-perfsonar-core/playbooks/perfsonar.yml
```

```
ansible-playbook \
  -i ansible-umich-perfsonar-core/inventory \
  --ask-pass \
  --ask-become-pass  \
  --ask-vault-pass \
  ansible-umich-perfsonar-core/playbooks/perfsonar.yml
```

---

** Some useful commands to troubleshoot the environment **

Display schedules on Testpoints, toolkits, maddash:
```
ansible ps-testpoints,ps-toolkits,ps-maddash \
  -i ansible-umich-perfsonar-core/inventory \
  --ask-pass --ask-become-pass \
  --ask-vault-pass \
  -a "psconfig remote list"
```

Display auth interfaces on Archivers:
```
ansible ps-archives \
  -i ansible-umich-perfsonar-core/inventory \
  --ask-pass --ask-become-pass  \
  -a "/usr/sbin/esmond_manage list_user_ip_address"
```

Delete an auth interface on Archivers:
```
ansible ps-archives \
  -i ansible-umich-perfsonar-core/inventory \
  --ask-pass --ask-become-pass  \
  -a "/usr/sbin/esmond_manage delete_user_ip_address USERNAME IPADDR"
```

Deploy AWS Testpoints:
```
ansible-playbook \
  -i ansible-umich-perfsonar-core/inventory \
  --ask-become-pass  \
  --limit 'testpoints-aws' \
  perfsonar.yml
```

Deploy Core Testpoints:
```
ansible-playbook \
  -i ansible-umich-perfsonar-core/inventory \
  --ask-pass --ask-become-pass  \
  --limit 'testpoints-core' \
  perfsonar.yml
```

Deploy PWA:
```
ansible-playbook \
  -i ansible-umich-perfsonar-core/inventory \
  --ask-pass --ask-become-pass  \
  --limit 'ps-psconfig-web-admin' \
  perfsonar.yml
```

Manage PWA users:
```
vi ansible-umich-perfsonar-core/inventory/group_vars/all/perfsonar/ps_pwa.yml
ansible-playbook \
  -i ansible-umich-perfsonar-core/inventory \
  --ask-pass --ask-become-pass  \
  --limit 'ps-psconfig-web-admin' --tags 'ps::pwa_users'\
  perfsonar.yml
```

Display PWA users:
```
ansible ps-psconfig-web-admin \
  -i ansible-umich-perfsonar-core/inventory \
  --ask-pass --ask-become-pass  \
  -a "docker exec -it sca-auth pwa_auth listuser --short"
```

Reset Password for PWA user from the command line:
```
ansible ps-psconfig-web-admin \
  -i ansible-umich-perfsonar-core/inventory \
  --ask-pass --ask-become-pass  \
  -a "docker exec -it sca-auth pwa_auth setpass \
  --username user --password x"
```

Delete PWA user from the command line:
```
ansible ps-psconfig-web-admin \
  -i ansible-umich-perfsonar-core/inventory \
  --ask-pass --ask-become-pass  \
  -a "docker exec -it sca-auth pwa_auth userdel \
  --username user"
```

Reset Password for PWA user interactively:
```
ansible-playbook \
  -i ansible-umich-perfsonar-core/inventory \
  --ask-pass --ask-become-pass  \
  roles/ansible-role-perfsonar-psconfig-web-admin/playbooks/pwa_passwd_reset.yml
```

Encrypt string

```
ansible-vault encrypt_string --stdin-name 'ansible_become_password'
```

Uninstall Old Docker Version

```
yum remove \
  docker \
  docker-client \
  docker-client-latest \
  docker-common \
  docker-latest \
  docker-latest-logrotate \
  docker-logrotate \
  docker-engine
```
Uninstall New Docker Version

```
yum remove \
  docker-ce \
  docker-ce-cli \
  docker-rhel-push-plugin \
  containerd.io
```
