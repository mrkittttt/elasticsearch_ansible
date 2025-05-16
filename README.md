# elasticsearch_ansible

Ansible version used -> 2.7.11

This repo contains ansible configurations to work with elasticsearch. Here you can deploy a cluster of any number of nodes or may have a single-node cluster.

Configurations contain tasks to install elasticsearch on Debian or Redhat based Linux distributions (For Redhat it has some tasks to open ports and change SELinux mode), create CA certs, and also transport certs to let elasticsearch nodes communicate with each other on 9300/tcp securely. Configurations yet does not set HTTP certs for port 9200/tcp, so communications on this port will be plain-text.

You may also easily scale your cluster and add more nodes to it by a single command which will be shown further

1. To start, hosts on which elasticsearch will be installed, are assumed to be in _elasticsearch_nodes_ group. So you have to set your inventory file as such. In an inventory file, you may also define future cluster's name in group variables

2. You must define password variables for certificates generation in *external_vars/secrets.yml*. Names of these variables are:
- _elastic_stack_ca_pass_
- _elastic_transport_pass_

Optionally, secure _secrets.yml_ with `ansible-vault encrypt secrets.yml` if necessary. Commands below assume that ansible-vault is not used

3. Set your remote user (if it's not the same as on host user) and private key in _ansible.cfg_. Otherwise, just run all commands adding options like `ansible-playbook --key-file ~/.ssh/id_rsa . . .`

4. Inside a main playbook - main.yml, a "master_node" host is described and there are particular tasks that are executed on this host. Basically, this is the first host described in your inventory. By a configuration, the first host will become a master node in elasticsearch cluster. Replace "master_node" according to your inventory


After making above pre-configurations, execute a playbook to start an elasticsearch cluster:
```bash
ansible-playbook main.yml
```

---
Whether you have a single host or multiple hosts, _/etc/elasticsearch/elasticsearch.yml_ is configured respectively. If you want to add more nodes after then, execute a playbook by skipping a creation of CA cert and resetting a password of "elastic" user. These tasks shall be done only once for your cluster:

```bash
ansible-playbook main.yml --skip-tags create_ca_cert,reset_elastic_pass
```

This will install elasticsearch on a new node and recreate transport certs. The new node will join cluster automatically, as elasticsearch monitors changes in SSL configs for around every 5 seconds

However, only if you firstly started a single-node cluster, and then you wanted to add more nodes to form a multi-node cluster, you shall restart elasticsearch on that node to let new nodes join an existing cluster

```bash
ansible master_node -m service -a "name=elasticsearch state=restarted" --become
```
