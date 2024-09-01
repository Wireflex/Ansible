# Playbooks
серия некоторых действий для выполнения и которые адресованы определенным наборам серверов, то есть базовые компоненты Ansible, которые записывают и исполняют конфигурацию Ansible

Ad-hoc команда ```ansible all -m ping -b``` в плейбуке будет выглядеть так:

playbookping.yml
```
---
- name: Test Connection to my servers
  hosts: all               # ansible all
  become: yes              # -b 

  tasks:

  - name: Ping my servers
    ping:                 # -m ping
```

Выполняем ```ansible-playbook playbookping.yml```

![image](https://github.com/user-attachments/assets/e2147baa-a8ae-472a-8201-91e139bd98fb)

Вместо state ( как в терраформ ) ansible юзает gathering facts и сравнивает текущее состояние с желаемым

Так же переделаем ```ansible all -m apt -a "name=apache2 state=present" -b``` и ```ansible all -m service -a "name=apache2 state=started enabled=yes" -b```

playbookapache2.yml
```
---
- name: Install default Apache Web Server
  hosts: all
  become: yes


  tasks:

  - name: Install Apache
    apt: name=apache2 state=latest

  - name: Start Apache and Enable
    service: name=apache2 state=started enabled=yes

```
Далее закинем на сервер кастомную страницу и настроим handler ( используется для выполнения действий в ответ на изменения, вызванные другими задачами в плейбуке )

playbookindex.yml
```
---
- name: Install Apache and Upload Web Page
  hosts: all
  become: yes


  vars:
    source_file: ./MyWebSite/index.html
    destin_file: /var/www/html

  tasks:
  - name: Install Apache
    apt: name=apache2 state=latest


  - name: Copy Web Page to servers
    copy: src={{ source_file }} dest={{ destin_file}} mode=0555
    notify: Restart Apache

  - name: Start WebServer and enable
    service: name=apache2 state=started enabled=yes


  handlers:
  - name: Restart Apache
    service: name=apache2 state=restarted
```
