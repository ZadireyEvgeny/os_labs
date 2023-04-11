<h1 align="center">Ansible</h1>
<h1>Установка Ansible</h1>
Ansible требует для своей работы Python 2.6 или выше.
Если необходимо, установите Ansible (yum install или apt install) и убедитесь что он корректно работает.
<h1>Подготовка стенда</h1>
Для начала работы нужно создать каталог ansible и поднять Vagrantfile
Для подключения к хосту nginx нам необходимо передать множество параметров. Мы можем узнать эти параметры с помощью vagrant ssh-config.
<h1>Inventory</h1>
В папке ansible создай файл inventory.
Сам inventory должен выглядеть примерно так: 
<pre>
[webservers]
nginx ansible_host=127.0.0.1 ansible_port=2222 ansible_private_key_file=/home/zadirei/os_lab/.vagrant/machines/nginx/virtualbox/private_key
</pre>
В этом файле стоит изменить параметры ansible_port и ansible_privat_key_file. Данную информацию можно посмотреть с помощью прошлой команды.
Чтобы убедиться, что что управляемый хост доступен, только без указания inventory файла: ansible -m ping nginx:
<pre>nginx | SUCCESS => {
 "ansible_facts": {
 "discovered_interpreter_python": "/usr/bin/python"
 },
 "changed": false,
 "ping": "pong"
}
</pre>
Если выводит не SUCCESS то ищи ошибку в inventory.
<h1>ansible.cfg</h1>
Чтобы не пришлось в дальнейшем каждый раз явно указывать наш инвентори файл в командной строке, создадим файл конфигурации ansible.cfg
создадим файл ansible.cfg со следующим содержанием:
<pre>
inventory = inventory
remote_user= vagrant
host_key_checking = False
transport=smart
</pre>
Теперь из inventory можно убрать информацию о пользователе:
<pre>
[webservers]
nginx ansible_host=127.0.0.1 ansible_port=2203 ansible_ssh_private_key_file=.vagrant/ma
chines/nginx/virtualbox/private_key
</pre>
Еще раз убедимся, что управляемый хост доступен, только теперь без явного указания
inventory файла: ansible -m ping nginx
<h1>Playbook epel</h1>
Напишем простой Playbook который будет выполнять установку пакета epel-release. Создаем файл epel.yml со следующим содержимым:
<pre>
- name: Install EPEL Repo
  hosts: webservers
  become: true
  tasks:
    - name: Install EPEL Repo package from standard repo
      yum:
        name: epel-release
        state: present
</pre>
После чего запустите выполнение Playbook командой: ansible-playbook epel.yml
<pre>
PLAY [Install EPEL Repo]
**************************************************************
TASK [Gathering Facts]
****************************************************************
ok: [nginx]
TASK [Install EPEL Repo package from standard repo]
***********************************
ok: [nginx]
PLAY RECAP
******************************************************************
**********
nginx : ok=2 changed=0 unreachable=0 failed=0 skipped=
0 rescued=0 ignored=0
</pre>
<h1>Шаблон</h1>
Далее создадим файл шаблона для конфига NGINX, имя файла nginx.conf.j2
<pre>
events {
 worker_connections 1024;
}
http {
 server {
   listen {{ nginx_listen_port }} default_server;
   server_name default_server;
   root /usr/share/nginx/html;
   location / {
   }
 }
}
</pre>
<h1>Handlers</h1>
В конце должен получиться файл nginx.yml:
<pre>
- name: Install nginx package from epel repo
  hosts: webservers
  become: true
  vars:
    nginx_listen_port: 8080
  tasks:
    - name: Install EPEL Repo package from standard repo
      yum:
        name: epel-release
        state: present
    - name: install nginx from repo
      yum:
        name: nginx
        state: latest
      notify:
        - restart ngihx
      tags:
         nginx-package
         packages
    - name: Create config file from template
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify:
        - restart nginx  
      tags:
        nginx-configuration
  handlers:
    - name: restart nginx
      systemd:
        name: nginx
        state: restarted
        enabled: yes 
</pre>
<h1>Проверка</h1>
Теперь проверяем на работу Nginx c помощью команды: vagrant ssh -c "ip addr show"
после выводв этой команды, нам нужно найти там публичный ip-адресс
<p>Затем в браузере откройте страницу http://192.168.11.150:8080</p>
192.168.11.150 - ваш публичный адрес
<p>Должно вывести так</p>
<img src="https://github.com/ZadireyEvgeny/ZadireyEvgeny/blob/main/Снимок%20экрана%20от%202023-04-10%2013-43-54.png">
