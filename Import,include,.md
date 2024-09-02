# Import vs Include
В Ansible `import` и `include` используются для включения одного файла в другой, но есть важные различия:

1. **import**:
   - Используется для статического включения файлов, таких как задачи, роли или playbooks.
   - Все импорты обрабатываются во время компиляции, то есть до выполнения плейбуков. Это позволяет проверить правильность и целостность при компиляции.
   - Поддерживает возможность импортировать задачи с фиксированными структурами, что помогает избежать проблем с порядком выполнения.

2. **include**:
   - Используется для динамического включения файла во время выполнения плейбука.
   - Задачи или файлы включаются в поток выполнения на этапе выполнения, что позволяет более гибко управлять выполнением на основе условий.
   - Подходит для условных включений, например, когда необходимо включать разные файлы в зависимости от значения переменной.
  
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
