# Loop
нужен для перечисления параметров, которые будут выдаваться в {{ item }} , 'item' зарезервированная переменная

будет записывать 'Z' в myfile.txt и выводить эту инфу с делеем 2 сек и максимум 10 раз, до тех пор,пока не напишет 4 раза( ZZZZ )

Loop и with_items - тож самое
```
---
- name: Loops playbook
  hosts: linux1
  become: yes

  tasks:
  - name: Say hello to all
    debug: msg="Hello {{ item }}"
    loop:
        - "Vasya"
        - "Petya"
        - "Dota"

  - name: loop until example
    shell: echo -n Z >> myfile.txt && cat myfile.txt
    register: output
    delay: 2
    retries: 10
    until: output.stdout.find("ZZZZ") == false

  - name: print output
    debug:
      var: output.stdout

  - name: install packages
    apt: name={{ item }} state=present
    with_items:
          - python
          - tree
          - mysql-client
```
'-n' не выводит новую строчку, т.е всё пишет в одной

Теперь сделаем копирование картинок из директории MyWebSite2 в апаче ( немного поправлен сам файл: блоки, рестарты и хендлеры )
```
---
- name: Install Apache and Upload Web Page
  hosts: all
  become: yes

  vars:
    source_folder: ./MyWebSite2
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

  - name: Copy Web Page to servers Debian
    copy: src={{ source_folder }}/{{ item }} dest={{ destin_folder }} mode=0555
    loop:
       - "index.html"
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
либо сделать по-нормальному
```
  - name: Copy Web Page to servers Debian
    copy: src={{ item }} dest={{ destin_folder }} mode=0555
    with_fileglob: "{{ source_folder }}/*.*"
```
