---
title: Ansible
---

### 变量

Ansible 是一套简单易用的自动化部署系统，唯独它的变量设计让我有些迷惑，因为有着太多地方和太多种方式来定义变量。

Ansible 有如下几种变量：定义在 Inventory 里的变量，定义在 Playbook 里的变量，由 included files 或 roles 加入进来的变量，以及 facts 等。

### Inventory 变量

一般在 `/etc/ansible/groups_vars/foo` 里定义。

### Playbook 变量

在定义一个 playbook 时指定变量：

	- hosts: webservers
	  vars:
		http_port: 80

这里的变量是 playbook 级的。注意，不能用这种方式为单个 task 定义变量，但 task 可以接受参数，见下文 Parameterization。

### Included files 中的变量

通过 include 指令可以将另外的 playbook 加入到该 playbook 中，其中的变量也一并引入。

### Roles 中的变量

如果一个 role 目录里有 `variables/main.yml` 文件，则里面的变量自动加入到 playbook 里。


### Parameteried tasks

其实也就是 modules arguments。

最简单的形式：

	---
	
	- hosts: webserver
	  vars: 
		http_port: 80
		max_clients: 200
	  remote_user: root
	  tasks:
		- name: ensure apache is at the latest version
		  yum: pkg=httpd state=latest
		- name: write the apache config file
		  template: src=/srv/httpd.j2 dest=/etc/httpd.conf
		  nofity:
			- restart apache
		- name: ensure apache is running(and enabled)
		  service: name=httpd state=started enabled=yes
	  handlers:
		- name: restart apache
		  service: name=httpd state=restarted

也可以用 YAML 字典的形式提供上述 `key=value` 参数：

	---
	...
	  tasks:
		- name: ensure apache is at the latest version
		- yum:
		  pkg: httpd
		  state: latest
	...

### Parameterized includes

可以向 included files 里传递变量：

	tasks:
	  - include: wordpress.yml wp_user=mellon
	  - include: wordpress.yml wp_user=collie
	  - include: wordpress.yml wp_user=mellon

传递的也可以是 list 或 dict 变量：

	tasks:
  	  - { include: wordpress.yml, wp_user: mellon, ssh_keys: ['keys/one.txt', 'keys/two.txt'] }

而后在 included files 里可以用 `\{\{ wp_user \}\}` 这种形式使用变量。

从 1.0 开始，还有另外一种传递变量的形式：

	tasks:
	  - include: wordpress.yml
		vars:
		  wp_user: mellon
		  some_list_variable:
			- alpha
			- beta
			- gamma


### Parameterized roles

向 roles 里传递变量：

	---

	- hosts: webservers
	  roles:
		- common
		- { role: foo_app_instance, dir: '/opt/a', port: 10080 }
		- { role: foo_app_instance, dir: '/opt/b', port: 10081 }

给 playbook 或者 task 打上标签，就可以选择执行一个大的 playbook 的一部分。

### Tag tasks

下面这种语法适用于为某个 play 或者 task 打标签：

	tasks:
	  - yum: name={{ item }} state=installed
	  	with_items:
		  - httpd
		  - memcached
	  	tags:
		  - packages

	  - template: src=templates/src.j2 dest=/etc/foo.conf
		tags:
		  - configuration

而后，可以选择执行或跳过该 playbook 的一部分：

	$ ansible-playbook example.yml --tags "packages"
	$ ansbile-playbook example.yml --skip-tags "configuration"

### Tag roles

为 role 里的每一个 task 设定 tags:

	---
	
	- hosts: webservers
	  roles:
		- { role: foo, tags: ["bar", "baz"] }

### Tag included files

为 included files 里的每一个 task 设定 tags：

	- include: foo.yml tags=web,foo

	
