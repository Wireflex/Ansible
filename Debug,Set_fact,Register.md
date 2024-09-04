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

Выполняем ```ansible-playbook playbookping.yml``` ( hosts можно переопределить в консоли ```ansible-playbook playbookping.yml -l linux1 ```

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
## Debug
используется для вывода отладочной информации во время выполнения плейбуков. Он позволяет выводить переменные, выражения и другую информацию, которая может быть полезна для отладки и проверки работы вашего плейбука
## Set_fact
позволяет устанавливать новые переменные или изменять значения существующих переменных в рамках выполнения плейбука
## Register
используется для сохранения результатов выполнения задачи в переменную, которая может быть использована в последующих задачах или условиях

```ansible linux1 -m setup``` содержит огромное кол-во переменных ( ansible_os_family,ipv4,ipv6 и еще миллион всего )

их так же можно задавать через debug, но уже без {{ }}

в hosts дописал переменную owner=Wireflex

```
---
- name: Variables
  hosts: all
  become: yes


  vars:
    message1: Privet
    message2: World
    secret  : kiojfisd3oj

  tasks:

  - name: Print Secret variables
    debug:
      var: secret

  - debug:
      msg: "Sekretnoe slovo: {{ secret }}"

  - debug:
      msg: "Vladelec etogo servera -->{{ owner }}<--"

  - set_fact: full_message="{{ message1}} {{ message2 }} from {{ owner }}"

  - debug:
      var: full_message

  - debug:
      var: ansible_distribution

  - shell: uptime
    register: results

  - debug:
      var: results

```
![image](https://github.com/user-attachments/assets/76b26335-00c8-4cc0-a817-09fbf84145c4)

можно посмотреть на Json и взять оттуда любую переменную ( stdout, например, который только время,дни,юзеров и LA показывает ), дописать 
```
  - debug:
      var: results.stdout
```
![image](https://github.com/user-attachments/assets/d476e663-6de9-495d-b83d-02f0411bce06)






