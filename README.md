# Installing ES, Kibana and Nginx with Ansible at Scale

Imagine you have a huge cluster to install ES on. More so all of the machines have similar characteristics of RAM, Storage type etc and you want to set the data directory separate from the root directory?
This set of simple scripts is for you.

## Please modify the scripts to meet your needs especially the `disk.yml` file!

# Preparation:
1. Install ansible first: 
    ``` sh
    $ sudo python -m pip install ansible
    ```
    [Must be installed on machine having connectivity to all other machines where one wants to install ES, Kibana, etc through SSH]
2. You may need to keep the private key for ssh handy in case it is used to access Machines (eg: In the case of AWS).  Let us name the key `your_key.pem`
3. Have IPs of all the systems handy and note down which systems will be your Master node, Data node and Coordinating Node. Also Nginx will be automatically installed onto your coordinating node system. If you do not want that remove `import-playbook nginx_main.yml and import-playbook nginx_install.yml` from the main.yml file.
4. This script uses pre-built ES roles. Run the following only after ansible has been installed:
    ``` sh
    $ ansible-galaxy install elastic.elasticsearch,7.9.1
    ```
    ``` sh
    $ ansible-galaxy install fedelemantuano.kibana
    ```
    ``` sh
    $ ansible-galaxy install jdauphant.nginx
    ```

# Changing Script values:
The following need to be checked and changed:
Remember the IP list you made earlier? It will help you now.

1. Input all IPs in the [inventory.ini](./inventory.ini) file in required positions (decide which machines would be your master node, data node, etc and put them under corresponding blocks). For example if 1.1.1.1 is master, 1.2.2.1 is coordinating, 1.3.3.1 is data and 1.2.2.1 is kibana, my inventory.ini file will look like this:
    ``` yaml
     [hosts]
      1.1.1.1
      1.2.2.1
      1.3.3.1
     [data]
      1.3.3.1
     [masters]
      1.1.1.1
     [coordinating]
      1.2.2.1
     [kibana]
      1.2.2.1
     ```
2. Continuing this example, the [elastic.yml](./elastic.yml) file must be modified as follows. 
    ``` yaml
    - hosts: masters
      roles:
       - role: elastic.elasticsearch
      vars:
       es_heap_size: "16g"
       es_data_dirs:
          - "/es-data/elasticsearch"
       es_config:
         cluster.name: "YOUR CLUSTER"
         network.host: 0
         cluster.initial_master_nodes: "1.1.1.1"
         discovery.seed_hosts: "1.1.1.1"
         http.port: 9200
         node.data: true
         node.master: true
         node.ingest: true
         node.ml: true
         cluster.remote.connect: false
         bootstrap.memory_lock: true
    - hosts: data
      roles:
        - role: elastic.elasticsearch
      vars:
        es_data_dirs:
          - "/es-data/elasticsearch"
        es_heap_size: "16g"
        es_config:
          cluster.name: "YOUR CLUSTER"
          network.host: 0
          discovery.seed_hosts: "1.1.1.1"
          http.port: 9200
          node.data: true
          node.master: false
          node.ml: false
          bootstrap.memory_lock: true
          indices.recovery.max_bytes_per_sec: 100mb
    - hosts: coordinating
      roles:
        - role: elastic.elasticsearch
      vars:
        es_heap_size: "16g"
        es_config:
          cluster.name: "YOUR CLUSTER"
          network.host: 0
          discovery.seed_hosts: "1.1.1.1"
          http.port: 9200
          node.data: false
          node.master: false
          node.ingest: false
          node.ml: false
          bootstrap.memory_lock: true
     ```
