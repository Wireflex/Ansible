# JinjaTemplate ( .j2 )
это шаблонизатор в ansible, позволяет генерировать файлы( странички html, к примеру, которые будут выдавать ansible_os_family,ip итд свои для каждого серва )

Берём свою переменную 'owner' из hosts + несколько дефолт переменных из ```ansible all -m setup```

<details> <summary>index.j2</summary>

```
<!DOCTYPE html>
<html>
<head>
        <title>dota</title>
</head>
<body>
<font color="gold">Owner of this Server is: {{ owner }}<br>
Server Host Name: {{ ansible_hostname }}<br>
Server OS family is: {{ ansible_os_family }}<br>
IP Adress is: {{ ansible_default_ipv4.address }}<br>
</body>
</html>
```
</details>

playbook.yml, не забываем при переносе поменять название с index.j2 на index.html
```
---
- name: Install Apache and Upload Web Page
  hosts: all
  become: yes

  vars:
    source_folder: ./MyWebSite2/index.j2
    destin_folder: /var/www/html

  tasks:

  - name: Check and print linux version
    debug: var=ansible_os_family

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
    template: src={{ source_folder }} dest={{ destin_folder }}/index.html mode=0555
    notify:
       - Restart Apache Debian
       - Restart Apache Redhat

  - name: Copy Web Page to servers Debian
    copy: src={{ source_folder }} dest={{ destin_folder }} mode=0555
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
