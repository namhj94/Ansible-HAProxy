# Ansible - Deploy different web service and HAProxy
### 목적
centos와 ubuntu vm에 web 서비스를 배포하고 haproxy노드에 haproxy서비스를 배포 하였다
### Abstract 
roles을 사용해 tasks를 분류하였으며, os를 분류하기 위해 facts를 사용함.<br>
jinja template과 매직변수인 hostvars를 사용하여 haproxy 설정파일의 백앤드서버의 ip를 동적으로 할당하였고, web서비스의 index.html의 내용또한 jinja template과 facts를 사용해 편집하였다.

## Setting

### Nodes
- PC OS: Ubuntu(Controller)
- host1 - CentOS7 (managed)
    - user, user
- host2 - CentOS7 (managed)
    - user, user
- host3 - Ubuntu 18.04 (managed)
    - devops, dkagh1.

### SSH key Authentication
```yaml
- inventory.ini

[centos]
host1 ansible_user=user ansible_password=user ansible_become_password=user
host2 ansible_user=user ansible_password=user ansible_become_password=user

[ubuntu]
host3 ansible_user=devops ansible_password=dkagh1. ansible_become_password=dkagh1.

- ansible.cfg

[defaults]
inventory=inventory.ini
host_key_checking=false

- key.yml
---
- hosts: all
  tasks:
  - name: Allow user to login any host with ansible's account
    authorized_key:
      user: user
      state: present
      key: "{{ lookup('file', '/home/student/.ssh/id_rsa.pub')}}"
    when: ansible_facts.distribution == "CentOS"
  - name: Allow user to login any host with ansible's account
    authorized_key:
      user: devops
      state: present
      key: "{{ lookup('file', '/home/student/.ssh/id_rsa.pub')}}"
    when: ansible_facts.distribution == "Ubuntu"
```

### Remove sudo password

```yaml
- inventory.ini

[centos]
host1 ansible_user=user ansible_password=user ansible_become_password=user
host2 ansible_user=user ansible_password=user ansible_become_password=user

[ubuntu]
host3 ansible_user=devops ansible_password=dkagh1. ansible_become_password=dkagh1.

- ansible.cfg

[defaults]
inventory=inventory.ini
host_key_checking=false
For CentOS
cp_cent.yml

---
- name: cp nopasswd file for no sudo passwd
  hosts: centos
  become: yes
  tasks:
    - copy:
        src: nopasswd
        dest: /etc/sudoers.d/

* nopasswd file
user ALL=(ALL) NOPASSWD: ALL

For Ubuntu
cp_ubuntu.yml
---
- name: cp nopasswd file for no sudo passwd
  hosts: ubuntu
  become: yes
  tasks:
    - copy:
        src: nopasswd_devops
        dest: /etc/suoers.d/

 * nopasswd_ubuntu
devops ALL=(ALL) NOPASSWD: ALL

완료 후 확인
- inventory.ini
[centos]
host1 ansible_user=user
host2 ansible_user=user

[ubuntu]
host3 ansible_user=devops

$ ansible all -a id -b
```
---

## Architecture
![images/example.png](images/example.png)

```bash
1. host1에 haproxy 서비스를 설치하세요.
2. haproxy의 설정 파일은 http://cccr3guro.iptime.org/ansible/EXAM/haproxy.cfg.j2
에서 다운로드 받아서 template 모듈로 배포하세요.
3. 다운로드 받은 템플릿 파일의 ????은 적당한 변수로 변경해야 합니다.
4. host2는 CentOS host3 Ubuntu입니다. 아파치 서비스를 각각 설치하고
index.html 에는 각 서버의 호스트네임이 들어가야합니다. 예를 들어 curl host2 를
하면 host2가 출력되어야 합니다.
5. 최종적으로 student$ curl host1 을 실행하면 host2와 host3이 교대로 출력되어야
합니다.
6. role을 사용해서 작성.
```
## Result
### codes
- haproxy role
  - [haproxy tasks](./roles/haproxy/tasks/main.yml)
  - [haproxy template](./roles/haproxy/templates/haproxy.cfg.j2)
- web role
  - [web tasks](./roles/web/tasks/)
  - [web template](./roles/web/templates/index.html)

### tree
```yaml
😕  exam git:(master) ✗ tree ~/Documents/Ansible/exam/
/home/student/Documents/Ansible/exam/
├── ansible.cfg
├── inventory.ini
├── roles
│   ├── haproxy
│   │   ├── defaults
│   │   │   └── main.yml
│   │   ├── files
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── meta
│   │   │   └── main.yml
│   │   ├── README.md
│   │   ├── tasks
│   │   │   └── main.yml
│   │   ├── templates
│   │   │   └── haproxy.cfg.j2
│   │   ├── tests
│   │   │   ├── inventory
│   │   │   └── test.yml
│   │   └── vars
│   │       └── main.yml
│   └── web
│       ├── defaults
│       │   └── main.yml
│       ├── files
│       ├── handlers
│       │   └── main.yml
│       ├── meta
│       │   └── main.yml
│       ├── README.md
│       ├── tasks
│       │   ├── main.yml
│       │   ├── setup-centos.yml
│       │   └── setup-ubuntu.yml
│       ├── templates
│       │   └── index.html
│       ├── tests
│       │   ├── inventory
│       │   └── test.yml
│       └── vars
│           └── main.yml
└── webservers_haproxy_role.yml
```
![images/result.png](images/result.png)
host1에서 haproxy 동작 확인
