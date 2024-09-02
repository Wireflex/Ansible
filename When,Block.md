# When (Условия)
позволяют выполнять задачи на основе определенных условий или переменных. Это позволяет создавать более гибкие и адаптивные плейбуки.
```
---
- name: Install Apache and Upload Web Page
  hosts: all
  become: yes


  vars:
    source_file: ./MyWebSite/index.html
    destin_file: /var/www/html

  tasks:
  - name: Check and print linux version
    debug: var=ansible_os_family

  - name: Install Apache for Debian
    apt: name=apache2 state=latest
    when: ansible_os_family == "Debian"

  - name: Install Apache for Redhat
    yum: name=httpd state=latest
    when: ansible_os_family == "Redhat"

  - name: Copy Web Page to servers
    copy: src={{ source_file }} dest={{ destin_file}} mode=0555
#    notify: Restart Apache

  - name: Start WebServer and enable for Debian
    service: name=apache2 state=started enabled=yes
    when: ansible_os_family == "Debian"

  - name: Start WebServer and enable for Redhat
    service: name=httpd state=started enabled=yes
    when: ansible_os_family == "Redhat"


  handlers:
  - name: Restart Apache Debian
    service: name=apache2 state=restarted

  - name: Restart Apache Redhat
    service: name=httpd state=restarted
```
соответственно, если один сервер на Debian, а другой на Redhat, установит апаче на оба и не выдаст ошибку, но если серверов штук 50, то влом писать везде when, поэтому можно юзать Blocks

# Blocks (Блоки)
позволяют группировать несколько задач вместе для управления ошибками и обработки исключений. Это полезно, когда вам нужно выполнить несколько задач и обработать ошибки, возникающие при выполнении этих задач.

```
---
- name: Install Apache and Upload Web Page
  hosts: all
  become: yes


  vars:
    source_file: ./MyWebSite/index.html
    destin_file: /var/www/html

  tasks:

  - name: Check and print linux version
    debug: var=ansible_os_family

  - block:  #=========Block for Debian=======#

    - name: Install Apache for Debian
      apt: name=apache2 state=latest

    - name: Start WebServer and enable for Debian
      service: name=apache2 state=started enabled=yes

    - name: Copy Web Page to servers Debian
      copy: src={{ source_file }} dest={{ destin_file}} mode=0555
      notify: Restart Apache Debian

    when: ansible_os_family == "Debian"


  - block: #==========Block for Redhat=======#

    - name: Install Apache for Redhat
      yum: name=httpd state=latest

    - name: Start WebServer and enable for Redhat
      service: name=httpd state=started enabled=yes

    - name: Copy Web Page to servers Redhat
      copy: src={{ source_file }} dest={{ destin_file}} mode=0555
      notify: Restart Apache Redhat

    when: ansible_os_family == "Redhat"


  handlers:
  - name: Restart Apache Debian
    service: name=apache2 state=restarted

  - name: Restart Apache Redhat
    service: name=httpd state=restarted
```
1 блок для Debian, 2ой для Redhat, 'when' пишем строго под 'block'


