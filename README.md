##brainstation-task2
## LXD Installation
```
sudo snap install lxd
```
```
Here need to assign storage pool name and flle system zfs also need to allocate store size (1GB Minimum) 

sudo lxd init

```
## container profile create
```
lxc profile create kubernetes
lxc profile edit kubernetes

# Insert the below configuration

config:
  boot.autostart: "true"
  linux.kernel_modules: ip_vs,ip_vs_rr,ip_vs_wrr,ip_vs_sh,ip_tables,ip6_tables,netlink_diag,nf_nat,overlay,br_netfilter
  raw.lxc: |
    lxc.apparmor.profile=unconfined
    lxc.mount.auto=proc:rw sys:rw cgroup:rw
    lxc.cgroup.devices.allow=a
    lxc.cap.drop=
  security.nesting: "true"
  security.privileged: "true"
description: ""
devices:
  aadisable:
    path: /sys/module/nf_conntrack/parameters/hashsize
    source: /sys/module/nf_conntrack/parameters/hashsize
    type: disk
  aadisable1:
    path: /sys/module/apparmor/parameters/enabled
    source: /dev/null
    type: disk
  aadisable3:
    path: /dev/kmsg
    source: /dev/kmsg
    type: disk
name: kubernetes
used_by: []
```
## Container Create

```
lxc launch --profile default --profile kubernetes ubuntu:20.04 master
lxc launch --profile default --profile kubernetes ubuntu:20.04 worker1
lxc launch --profile default --profile kubernetes ubuntu:20.04 worker2
lxc launch --profile default --profile kubernetes ubuntu:20.04 worker3
```

##Before run ansible commmand need to do few things .

##Login Every LXC Server and set the root password 

#ssh premission root from outside
```
```
lxc exec master bash ;login every node
passwd root          ;set password


vi /etc/ssh/sshd_config
PermitRootLogin yes  
PasswordAuthenicaiton yes
```

# login to the container to set password.
 lxc exec master bash
 ..
 ..

## Creating Kubernetes cluster using Ansible

##From Host Machine need to run ssh-keygen command before run ansible command. Other wise you will face unauthorized error or need to type every time password by using -k parameter.
```
ssh-keygen   
```

```
ansible-playbook add-users.yaml
ansible-playbook install-prerequsites-k8s.yaml
ansible-playbook install-master.yaml
ansible-playbook join-workers.yaml
```

## Kubernetes Dashboard and metric server
```
kubectl create -f dashboard-admin-user.yaml

kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.0/aio/deploy/recommended.yaml
kubectl patch -n kubernetes-dashboard svc kubernetes-dashboard --type='json' -p '[{"op":"replace","path":"/spec/type","value":"NodePort"}]'

# get the token to login the dashboard
kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"


kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```


