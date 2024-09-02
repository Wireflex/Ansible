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
