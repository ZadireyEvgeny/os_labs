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
в этом файле стоит изменить параметры ansible_port и ansible_privat_key_file. Данную информацию можно посмотреть с помощью прошлой команды.
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
