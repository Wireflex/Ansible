# Role

представляет собой структурированный способ организации задач, переменных, шаблонов и других файлов, связанных с конкретной функциональностью или компонентом системы. Роли позволяют повторно использовать код и упрощают управление конфигурацией

![image](https://github.com/user-attachments/assets/0a0135f0-befa-4d7f-8b79-a4d2509cc201)

- tasks: содержит файлы с задачами, которые выполняются в рамках роли.
- handlers: содержит файлы с обработчиками, которые могут быть вызваны после выполнения задач.
- templates: содержит шаблоны файлов, которые могут быть скопированы на целевой хост и настроены с использованием переменных.( .j2 )
- files: содержит статические файлы, которые могут быть скопированы на целевой хост без изменений.
- meta: содержит файлы с метаданными о роли, такими как зависимости от других ролей.
- tests: содержит файлы для тестирования роли.

- var: Хранит переменные, которые могут быть заменены на конкретные значения во время выполнения плейбука. Например, IP-адрес сервера, имя пользователя или версия пакета ( переопределит переменную из defaults )
- default: Хранит дефолтные значения для переменных. Если переменная не определена в `var` или при запуске плейбука, то Ansible будет использовать значение из `default` ( приоритет ниже чем у vars )

Таким образом, `var` - это более гибкий вариант, позволяющий использовать разные значения переменных в разных ситуациях, а `default` - это безопасность и удобство, когда вы хотите задать базовые значения

Роль создаётся командой ```ansible-galaxy init название_роли```

![image](https://github.com/user-attachments/assets/8b2df6e4-825d-4ff4-8dff-ccd633436192)

Грубо говоря, нужно, к примеру, этот плейбук

<details> <summary>playbook.yml</summary>

```
---
- name: Install Apache and Upload Web Page
  hosts: all
  become: yes

  vars:
    source_folder: ./MyWebSite2
    destin_folder: /var/www/html

  tasks:

  - block:  #=========Block for Debian=======#

    - name: Install Apache for Debian
      apt: name=apache2 state=latest

    - name: Start WebServer and enable for Debian
      service: name=apache2 state=started enabled=yes

    when: ansible_os_family == "Debian"

  - block: #==========Block for Redhat=======#

    - name: Install Apache for Redhat
      yum: name=httpd state=latest

    - name: Start WebServer and enable for Redhat
      service: name=httpd state=started enabled=yes

    when: ansible_os_family == "Redhat"

  - name: Generate J2
    template: src={{ source_folder }}/index.j2 dest={{ destin_folder }}/index.html mode=0555
    notify:
       - Restart Apache Debian
       - Restart Apache Redhat

  - name: Copy Web Page to servers Debian
    copy: src={{ source_folder }}/{{ item }} dest={{ destin_folder }} mode=0555
    loop:
       - "dota.png"
       - "dota2.png"
       - "dota3.png"
    notify:
       - Restart Apache Debian
       - Restart Apache Redhat

  handlers:
  - name: Restart Apache Debian
    service: name=apache2 state=restarted
    when: ansible_os_family == "Debian"

  - name: Restart Apache Redhat
    service: name=httpd state=restarted
    when: ansible_os_family == "Redhat"

```
</details>

распределить по файлам в директориях роли, а в самом плейбуке оставить только hosts и название ролей

3 картинки из ```MyWebSite2/``` просто переносим в ```roles/deploy_apache_web/files/```

Сгенерированный шаблон index.j2 в ```roles/deploy_apache_web/templates/```

Переменная ```source_folder: ./MyWebSite2```больше не нужна,т.к ансибл знает откуда брать пикчи, а переменную ```destin_folder: /var/www/html``` переносим либо в ```roles/deploy_apache_web/vars```, либо в

<details> <summary>roles/deploy_apache_web/defaults/main.yml</summary>

```
---
# defaults file for deploy_apache_web

destin_folder: /var/www/html
```
</details>

2 хендлера из конца файла в 

<details> <summary>roles/deploy_apache_web/handlers/main.yml</summary>

```
---
# handlers file for deploy_apache_web

- name: Restart Apache Debian
  service: name=apache2 state=restarted
  when: ansible_os_family == "Debian"

- name: Restart Apache Redhat
  service: name=httpd state=restarted
  when: ansible_os_family == "Redhat"
```
</details>

и tasks: в

<details> <summary>roles/deploy_apache_web/tasks/main.yml</summary>

```
---
# tasks file for deploy_apache_web

- block:  #=========Block for Debian=======#

  - name: Install Apache for Debian
    apt: name=apache2 state=latest

  - name: Start WebServer and enable for Debian
    service: name=apache2 state=started enabled=yes

  when: ansible_os_family == "Debian"

- block: #==========Block for Redhat=======#

  - name: Install Apache for Redhat
    yum: name=httpd state=latest

  - name: Start WebServer and enable for Redhat
    service: name=httpd state=started enabled=yes

  when: ansible_os_family == "Redhat"

- name: Generate J2
  template: src=index.j2 dest={{ destin_folder }}/index.html mode=0555
  notify:
      - Restart Apache Debian
      - Restart Apache Redhat

- name: Copy Web Page to servers Debian
  copy: src={{ item }} dest={{ destin_folder }} mode=0555
  loop:
      - "dota.png"
      - "dota2.png"
      - "dota3.png"
  notify:
      - Restart Apache Debian
      - Restart Apache Redhat
```
</details>

новый playbook.yml
```
---
- name: Install Apache and Upload Web Page
  hosts: all
  become: yes

  roles:
    - { role: deploy_apache_web, when: ansible_system == 'Linux' }
#    - deploy_db
```
можно юзать сразу несколько ролей ( неактуальные можно комментить, а потом раскомменчивать, когда нужно юзать )
