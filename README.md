![image](https://github.com/Wireflex/Ansible/assets/165675775/a0566b9d-8065-4fda-9516-6b08c06500fa)

это инструмент автоматизации, который используется для управления конфигурацией, развертывания приложений и выполнения задач в инфраструктуре

2 метода управления конфигурацией:
 - Pull - на управляемых серверах установлен Agent, который делает Pull-request настроек от Master ( Chef, Puppet, SaltStack )
 - Push - на управляемых серверах не нужно ничего устанавливать, Master сам делает Pull настроек ( Ansible )

![image](https://github.com/user-attachments/assets/12987903-a769-4262-95c9-b1b8d9cf803b)

<details> <summary>Установка</summary>

```
sudo apt-add-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible
```
</details>

## hosts/inventory
список хостов, которыми можно управлять через Ansible, их можно по-всякому группировать

```
[stage_group]
linux1 ansible_host=18.199.164.53

[prod_group]
linux2 ansible_host=192.168.0.70

```

```ansible-inventory --list```

Здесь linux1 сервер поднят в AWS, приатачен приватный ключ 'wireflex-key-frankfurt.pem', поэтому без пароля, ssh-copy-id, обменом публичных ключей итд

В случае с linux2 нужно ввести пароль

Либо же заранее установить ssh-соединение с сервером каким-либо образом ( пароль, ключ )

---

Ad-hoc команды - это возможность запустить какое-то действие Ansible из командной строки

```ansible -i hosts linux1 -m ping``` -i (инвентарь, у нас это hosts), linux1 (конкретный сервер, можно было выбрать все серверы 'all', или к примеру, группу prod), -m (модуль, по сути в ансибл всё модули, тут просто проверка ping-pong)

![image](https://github.com/user-attachments/assets/a113f618-4402-4f8f-a9f8-fc58efebdbfb)

[Все модули](https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html) либо 'ansible (apt) в инете

[Самые важные-1](https://habr.com/ru/companies/slurm/articles/707130/)

[Самые важные-2](https://habr.com/ru/companies/slurm/articles/707986/)

## ansible.cfg

файл конфигурации Ansible, позволяет настроить различные аспекты работы Ansible
<details> <summary>такие как</summary>

```
* Пути к инвентарю: Указывает, где искать файлы инвентаря, если они не находятся в стандартном месте (`/etc/ansible/hosts`).
* Пути к playbook: Указывает, где искать playbook, если они не находятся в текущей директории.
* Параметры по умолчанию: Устанавливает значения по умолчанию для различных параметров, таких как:
    * `ansible_user`: Пользователь для входа на хосты.
    * `ansible_port`: Порт для подключения к хостам.
    * `ansible_python_interpreter`: Путь к интерпретатору Python.
* Логирование: Настраивает уровень детализации логов.
* Прокси: Настраивает использование прокси-сервера.
```
</details>

А так же:
* Глобальная конфигурация: `ansible.cfg` обычно находится в `/etc/ansible/ansible.cfg` и действует для всех команд Ansible.
* Локальная конфигурация: Вы также можете создать `ansible.cfg` в текущей директории, чтобы переопределить глобальные настройки.
* Дополнительные настройки: Можно использовать аргументы командной строки, например, `-i` для указания другого инвентаря или `-m` для выбора модуля.

```
[defaults]
inventory         = ./hosts
host_key_checking = False
```

А для переменных( ключей/паролей итд) лучше создать отдельную директорию group_vars, и в неё закинуть переменные для stage_group и prod_group

<details> <summary>/group_vars/stage_group</summary>

```
---
ansible_user:                 ubuntu
ansible_ssh_private_key_file: /home/wireflex/.ssh/wireflex-key-frankfurt.pem

```
</details>

<details> <summary>/group_vars/prod_group</summary>

```
---
ansible_user:     wireflex
ansible_ssh_pass: dota228
```
</details>

Теперь команда выглядит так ```ansible all -m ping```

![image](https://github.com/user-attachments/assets/cfaee11c-b562-48c4-bc88-bd2caf237f85)

```ansible all -m shell -a "uptime"``` ( -a аргумент, вместо 'shell' можно юзать 'command', но там не работают переменные,>,<,| итд)

![image](https://github.com/user-attachments/assets/9eb9b82e-fedd-4d03-9d2c-7d3f0a2bdf64)

```ansible linux1 -m setup``` инфа о серве

Можно создать файл 'hello' и передать на сервы ```ansible all -m copy -a "src=hello dest=/home/ mode=0777" -b``` -b это sudo

и затем удалить его ```ansible all -m file -a "path=/home/hello state=absent" -b```

скачать что-то из инета ```ansible all -m get_url -a "url=https://dota3.ru dest=/home" -b```

проверить подключение ```ansible all -m uri -a "url=http://www.dota3.ru"``` и вывести его, дописать ```return_content=yes"```

установить ```ansible all -m apt -a "name=apache2 state=present" -b```

включить и добавить а автозагрузку```ansible all -m service -a "name=apache2 state=started enabled=yes" -b```

инфа о файле ```ansible all -m shell -a "ls /var" -v``` чем больше -vvvv, чем больше инфы
