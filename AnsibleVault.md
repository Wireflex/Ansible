# AnsibleVault
инструмент для шифрования конфиденциальной информации, такой как пароли, ключи и другие секреты, используемые в плейбуках Ansible. Это позволяет безопасно хранить конфиденциальные данные в репозитории кода или передавать их по сети.

```ansible-vault create mysecret.txt``` создание зашифрованного файла(нужно придумать пароль и дальше он будет его спрашивать) ```ansible-vault rekey mysecret.txt``` поменять пароль

открывается вонючий vim, пишем 'Hello' ( Esc, :wq!, чтоб выйти )

![image](https://github.com/user-attachments/assets/1de5a343-d101-41a2-8f09-f8ac555c20fb)

```ansible-vault view mysecret.txt``` посмотреть ( вместо cat )

```ansible-vault edit mysecret.txt``` отредактировать файл

```ansible-vault encrypt playbookvault.yml``` зашифровать уже созданный файл ( с этого момента взаимодействовать, очевидно, можно только через ansible-vault [view,edit итд] и запускать обычным способом нельзя )

```ansible-vault decrypt playbookvault.yml``` дешифровка файла

```ansible-playbook playbookvault.yml --ask-vault-pass``` запустить плейбук, введя пароль

можно записать пароль в файл и запустить уже без ввода пароля ```ansible-playbook playbookvault.yml --vault-password-file mypass.txt```

<details> <summary>playbook_vault.yml</summary>

```
---
- name: vault
  hosts: all
  become: yes

  vars:
    admin_pass: DS95FD3D31

  tasks:
  - name: pass
    copy:
      dest: "/home/pass.txt"
      content: |
        password= {{ admin_pass }}

```
</details>

Можно шифровать не весь файл, а только определенную строчку ( переменную ), например, пароль

```ansible-vault encrypt_string``` вводим пароль, который хотим зашифровать, ctrl-d два раза, и копируем всё, начиная от !vault,  и вставляем прям в плейбук вместо НЕзашифрованного пароля

![image](https://github.com/user-attachments/assets/1ab25622-4f4c-497e-8ddb-3719ffdd775c)

если попытаться запустить плейбук обычным способом выдаст ошибку

![image](https://github.com/user-attachments/assets/d29b92ea-3dba-488a-98f5-90fe00185b2a)

```ansible-playbook playbookvault.yml --ask-vault-pass``` запускаем с vault-паролем

и на целевом сервере он уже дешифруется

![image](https://github.com/user-attachments/assets/3dd29307-cfae-489d-a3f6-8571e9c1dc39)

```echo -n "dota2" | ansible-vault encrypt_string``` чуть удобнее, только придумать пароль
