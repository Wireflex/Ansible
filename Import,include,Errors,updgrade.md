# Import vs Include
В Ansible `import` и `include` используются для включения одного файла в другой, но есть важные различия:

▎import

• Статическая загрузка: Все файлы, которые вы импортируете с помощью import, загружаются во время парсинга playbook. Это значит, что они становятся частью основного playbook и выполняются в том порядке, в котором они были импортированы.

• Используется для: Импортирования задач, ролей, плейбуков и т.д., когда вам нужно, чтобы все содержимое было известно заранее.

• Пример:
    
    - import_tasks: tasks.yml
    

▎include

• Динамическая загрузка: Файлы, которые вы включаете с помощью include, загружаются во время выполнения. Это позволяет более гибко управлять логикой выполнения, например, включать разные задачи в зависимости от условий.

• Используется для: Включения задач, когда вам нужно принимать решения во время выполнения (например, на основе переменных или условий).

• Пример:
    
    - include_tasks: tasks.yml
      when: some_condition
    

▎Основные отличия:

1. Когда выполняется:

   • import — на этапе парсинга.

   • include — на этапе выполнения.

2. Гибкость:

   • import менее гибок, так как все загружается сразу.

   • include позволяет динамически выбирать, что выполнять.
  
Родительские директории так же создадутся
<details> <summary>create_folders.yml</summary>

```
---
- name: create folder1
  file:
    path: /home/secret/folder1
    state: directory
    mode: 0755

- name: create folder2
  file:
    path: /home/secret/folder2
    state: directory
    mode: 0755

```
</details>

Переменную mytext можно переопределить через --extra-var , либо дописав в playbook.yml другую возле нужного файла
<details> <summary>create_files.yml</summary>

```
---
- name: create file1
  copy:
    dest: /home/secret/file1.txt
    content: |
      hello {{ mytext }}

- name: create file2
  copy:
    dest: /home/secret/file2.txt
    content: |
      hello2 {{ mytext }}

```
</details>


<details> <summary>playbook.yml</summary>

```
---
- name: import
  hosts: all
  become: yes

  vars:
    mytext: "dota"

  tasks:
  - name: ping
    ping:

  - include_tasks: create_folders.yml
  - import_tasks: create_files.yml
```
</details>

# Errors
```ignore_errors: yes``` позволяет игнорировать ошибку и продолжать выполнение плейбука

```failed_when:``` наоборот будет выдавать ошибку, если было соблюдено условие ( мы печатаем hello World, а он ищет World в выводе results как раз )

закомменченный return code == 0 так же выдаст ошибку, когда команда выполнится успешно О_о

```any_errors_fatal: true``` сразу закончит плейбук, если будет ошибка, однако ```ignore_errors: yes``` имеет приоритет, и остальные таски всё равно продолжат выполняться
```
---
- name: dota
  hosts: all
  any_errors_fatal: true
  become: yes

  tasks:
    - name: Install package (treeee)
      apt:
        name: treeee
        state: latest
      ignore_errors: yes

    - name: Execute shell command
      shell: echo Hello World
      register: results
#      failed_when: results.rc == 0
      failed_when: "'World' in results.stdout"

    - debug:
        var: results

    - name: Execute another shell command
      shell: echo Privet

```

## Update + upgrade

```
  - name: update
    apt:
      update_cache: yes

  - name: upgrade
    apt:
      upgrade: yes
```
