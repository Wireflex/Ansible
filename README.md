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

[all_linux:children]
stage_group
prod_group

```

```ansible-inventory --list```

Здесь linux1 сервер поднят в AWS, приатачен приватный ключ амазона

В случае с linux2 нужно ввести пароль

Либо же заранее установить ssh-соединение с сервером каким-либо образом ( пароль, ключ )

---

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
host_key_checking = False   # чтоб fingerprint не тыкать
```

## group_vars

А для переменных( ключей/паролей итд) лучше создать отдельную директорию group_vars, и в неё закинуть переменные (к примеру, для stage_group и prod_group )

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
ansible_become_pass: dota   # либо у юзера вообще убрать пароль в visudo (%sudo   ALL=(ALL:ALL) ALL
                                                                        # wireflex ALL=(ALL:ALL) NOPASSWD:ALL)
```
</details>

[Все модули](https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html) либо "ansible (apt)" в инете

[Самые важные-1](https://habr.com/ru/companies/slurm/articles/707130/)

[Самые важные-2](https://habr.com/ru/companies/slurm/articles/707986/)

---

можно задать внешние переменные ```ansible-playbook playbookrole.yml --extra-var "MYHOSTS=stage_group"``` extra-var переопределит переменные в файлах 

<details> <summary>playbook.yml</summary>

```
---
- name: Install Apache and Upload Web Page
  hosts: "{{ MYHOSTS }}"
  become: yes

  roles:
    - { role: deploy_apache_web, when: ansible_system == 'Linux' }
```
</details>

