# Ansible



## Get Started

- [Install on Debian](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-debian)

### Install on macOS

```bash
# 较旧版本的 Linux 请使用 8.7 版本
# 否则使用最新版!!!
$ sudo pip install ansible==8.7

$ ansible --version           
ansible [core 2.15.9]
  config file = /Users/linnzh/.ansible.cfg
  configured module search path = ['/Users/linnzh/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /Users/linnzh/anaconda3/lib/python3.10/site-packages/ansible
  ansible collection location = /Users/linnzh/.ansible/collections:/usr/share/ansible/collections
  executable location = /Users/linnzh/anaconda3/bin/ansible
  python version = 3.10.9 (main, Mar  1 2023, 12:20:14) [Clang 14.0.6 ] (/Users/linnzh/anaconda3/bin/python)
  jinja version = 3.1.2
  libyaml = True
  
# 查看帮助
$ ansible-doc file
$ ansible-doc shell

# 查看已安装的模块
$ ansible-doc -l

# 设置: https://github.com/ansible/ansible/issues/76322
$ export OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES
```

See: [Get started](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

### Add Remote server

```bash
# link: https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html

local:
  hosts:
    app:
      ansible_host: 192.168.31.129
    redis:
      ansible_host: 192.168.31.130
  vars:
    ansible_ssh_private_key_file: ~/.ssh/id_rsa

test:
  hosts:
    monitor:
      ansible_host: 18.236.135.174
  vars:
    ansible_ssh_user: ubuntu
    ansible_python_interpreter: /usr/bin/python3

all:
  vars:
    ansible_connection: ssh
    ansible_ssh_user: root
    ansible_port: 22
    ansible_ssh_private_key_file: ~/.ssh/id_server
    ansible_ssh_pipelining: True
    ansible_python_interpreter: /usr/bin/python3
```

Test connection:

```bash
# ping 名为 local 的主机组下的主机
$ ansible local -m ping
192.168.31.129 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.31.130 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

### Example

#### Write a file

```bash
# 使用 file 模块向 local 主机组执行一条文件命令（写文件）
$ ansible local -m file -a "path=~/ansible-module-file.txt state=touch group=www-data owner=www-data"
192.168.31.129 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "dest": "/root/file-module-by-ansible.txt",
    "gid": 33,
    "group": "www-data",
    "mode": "0644",
    "owner": "www-data",
    "size": 0,
    "state": "file",
    "uid": 33
}
192.168.31.130 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "dest": "/root/file-module-by-ansible.txt",
    "gid": 33,
    "group": "www-data",
    "mode": "0644",
    "owner": "www-data",
    "size": 0,
    "state": "file",
    "uid": 33
}

# 创建目录
# 使用 file 模块也可以创建目录,与执行 mkdir -p 效果类似
$ ansible local -m file -a "path=~/ansible-module-file state=directory group=www-data owner=www-data mode=755"

# 删除目录（递归）
$ ansible local -m file -a "dest=~/ansible-module-file state=absent"
```

#### Execute shell

```bash
$ ansible local -m shell -a "echo 'Hello, this is ansible.' >> ~/ansible-module-shell.txt"
192.168.31.129 | CHANGED | rc=0 >>

192.168.31.130 | CHANGED | rc=0 >>
```

#### Other

```bash
# 删除账户
$ ansible local -m user -a "name=ansible1 state=absent"

# 创建账户
$ ansible local -m user -a "name=ansible1 password=<crypted password here>"

# git 拉取代码
$ ansible local -m git -a "repo=git://foo.example.org/repo.git dest=/srv/myapp version=HEAD"

# More: https://ansible-tran.readthedocs.io/en/latest/docs/intro_adhoc.html
```



#### Playbook

```yaml
# path: playbooks/example.yml
# 语法检查：ansible-playbook -C example.yml
- name: create a file and append content at remote server

  # 指定主机清单分组名
  hosts: local
  gather_facts: true

  # 定义任务（task）
  tasks:
    # Task #1 使用 file 模块创建一个空文件
    - name: create a empty file at remote server
      file:
        state: touch
        group: www-data
        owner: www-data
        path: ~/ansible-example-file.txt
    # Task #2 使用 shell 模块对文件进行修改
    - name: append lines of content
      shell: |
        echo 'Hello
        Ansible
        Come from Ansible' >> ~/ansible-example-shell.txt
```

Execute check：

```bash
$ ansible-playbook -C playbooks/example.yml 

PLAY [create a file and append content at remote server] ******************************************

TASK [Gathering Facts] ****************************************************************************
ok: [192.168.31.129]
ok: [192.168.31.130]

TASK [create a empty file at remote server] *******************************************************
changed: [192.168.31.129]
changed: [192.168.31.130]

TASK [append lines of content] ********************************************************************
skipping: [192.168.31.130]
skipping: [192.168.31.129]

PLAY RECAP ****************************************************************************************
192.168.31.129             : ok=2    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
192.168.31.130             : ok=2    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0 
```

Execute playbook:

```bash
# 输出和 check 一致
$ ansible-playbook playbooks/example.yml
```

### Jinjia2 Template

```yaml
# 将 nginx 配置文件使用模板进行替换
- name: substitue the nginx configuration

  # 指定主机清单分组名
  hosts: nginx
  gather_facts: true

  tasks:
    # 使用 debug 展示 ansible 收集的主机信息
    - name: display ansible_facts variable
      debug:
        var: ansible_facts
    # 使用 jinjia2 模板替换
    - name: substitue
      template:
        src: j2/nginx_conf.j2
        dest: ~/nginx.conf
      # nginx 主机组下的每个主机都执行该任务
      with_items: "{{ groups['nginx'] }}"
```

J2 template：

```nginx
user www-data;

worker_processes 2;
worker_cpu_affinity auto;
worker_rlimit_nofile 1024;

daemon on;
master_process on;

lock_file logs/nginx.lock;
pid log/nginx.pid;
error_log log/nginx-error.log error;

events {
    use epoll;
    worker_connections 1024;
    worker_aio_requests 32;
    multi_accept on;
    accept_mutex off;
    accept_mutex_delay 500ms;
}

http {
    include /usr/local/nginx/conf/mime.types;
    access_log logs/nginx-access.log;

    server {
        server_name {{ server_host }};
        listen *:{{ server_port }};

        location / {
            root html
            index index.html;
        }
    }
}
```

## Prometheus

Reference

- [Galaxy - Prometheus](https://galaxy.ansible.com/ui/repo/published/prometheus/prometheus/)
- [Node exporter role](https://prometheus-community.github.io/ansible/branch/main/node_exporter_role.html)



### Node Exporter

```bash
# 安装 prometheus 集合
# 里面包含了各种 exporter
$ ansible-galaxy collection install prometheus.prometheus --force
```

Playbook:

```yaml
- name: Install library
  hosts: local
  tasks:
    # 应该在运行 ansible 的机器上安装 gnu-tar 的
    # Example: brew install gnu-tar
    - name: install unzip
      shell: sudo apt install -y unzip tar

- name: Install and configure node_exporter
  hosts: all
  become: yes
  gather_facts: true
  tags:
    - metric
  roles:
    # link: https://prometheus-community.github.io/ansible/branch/main/node_exporter_role.html
    - name: prometheus.prometheus.node_exporter
      vars:
        node_exporter_web_listen_address: "0.0.0.0:9100"
        node_exporter_system_group: "node-exp"
        node_exporter_system_user: "node-exp"
        node_exporter_version: "1.7.0"
        node_exporter_web_telemetry_path: "/metrics"
```

Run：

```bash
# 执行 playbook 脚本（--step，按步骤执行）
$ ansible-playbook playbooks/prometheus.yml --step

# 可以看到 node_exporter 的输出
$ curl -X GET cache.linnzh.com:9100/metrics
```

### Prometheus

Playbook：

```yaml
---

# 在最开始收集系统信息
- name: Gather facts
  hosts: all
  gather_facts: yes

- name: Install prometheus
  hosts: all
  become: yes
  tags:
    - monitor
  roles:
    # link: https://prometheus-community.github.io/ansible/branch/main/prometheus_role.html
    - name: prometheus.prometheus.prometheus
      vars:
        prometheus_web_listen_address: "0.0.0.0:9090"
        prometheus_system_group: "monitor"
        prometheus_system_user: "monitor"
        prometheus_config_dir: "/etc/prometheus"

- name: Install and configure node_exporter
  hosts: all
  become: yes
  tags:
    - exporter
  roles:
    # link: https://prometheus-community.github.io/ansible/branch/main/node_exporter_role.html
    - name: prometheus.prometheus.node_exporter
      vars:
        node_exporter_web_listen_address: "0.0.0.0:9100"
        node_exporter_system_group: "node-exp"
        node_exporter_system_user: "node-exp"
        node_exporter_version: "1.7.0"
        node_exporter_web_telemetry_path: "/metrics"

- name: Install and configure redis_exporter
  hosts: all
  become: yes
  tags:
    - exporter
  roles:
    # link: https://prometheus-community.github.io/ansible/branch/main/redis_exporter_role.html
    - name: prometheus.prometheus.redis_exporter
      vars:
        redis_exporter_web_listen_address: "0.0.0.0:9121"
        redis_exporter_addr: "redis://localhost:6379"
        redis_exporter_log_format: "text"
        redis_exporter_password: ""
        redis_exporter_system_group: "monitor"
        redis_exporter_system_user: "monitor"
        redis_exporter_version: "1.58.0"
        redis_exporter_web_telemetry_path: "/metrics"
```

Run:

```bash
$ ansible-playbook playbooks/prometheus.yml
```

## Command Options

### Playbook

#### 运行时指定主机

而不是在 playbook 中硬编码主机组。

```bash
# 使用 -l 指定主机组（否则使用 all）
$ ansible-playbook playbooks/prometheus.yml -l test
```

#### 使用 tags 指定运行的步骤

而不是全部执行。

```bash
$ ansible-playbook playbooks/prometheus.yml --tags "exporter" -vvv
```

