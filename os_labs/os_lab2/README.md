<h1>Задание</h1>
<pre>
1. На серверах rrobin, web1, web2 установить nginx.
2. На серверах web1, web2 Nginx должен работать по порту 8080 и отдавать кастомную страницу, зайдя на которую можно понять на каком сервере вы находитесь.
3. На сервере rrobin Nginx должен обеспечить балансировку нагрузки серверов web1 и web2 в режиме round-robin. Вес каждого сервера одинаковый.
4. Установка и настройка всего ПО должна быть обеспечена Ansible-сценарием.
</pre>
<h1>Начало</h1>
Перед поднятием вагранта вам требуется изменить сам Vagrantfile, a именно в  vagrantfile в строчке<br><b>ssh_pub_key = File.readlines("/home/zadirei/.ssh/id_rsa.pub").first.strip</b><br>
вместо <em>zadirei</em> подставить свое имя пользователя.<br>
Когда изменили Vagrantfile теперь можно проднимать сам vagrant
<pre>vagrant up</pre>
Следующим действием мы вводим команду ansible-playbook nginx.yml
Должно быть примерно так:
<img src="https://github.com/ZadireyEvgeny/ZadireyEvgeny/blob/main/3.png">
Сейчас можно перейти http://192.168.11.113 и проверить работает ли балансировщик.
<p>По идее у вас должно получиться следующее:</p>
<img src="https://github.com/ZadireyEvgeny/ZadireyEvgeny/blob/main/1.png">
При обновлении сраницы через shift + f5  у вас должна появиться другая страница:
<img src="https://github.com/ZadireyEvgeny/ZadireyEvgeny/blob/main/2.png">
