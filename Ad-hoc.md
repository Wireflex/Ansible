# Ad-hoc
это возможность запустить какое-то действие Ansible из командной строки

```ansible -i hosts linux1 -m ping``` -i (инвентарь, у нас это hosts), linux1 (конкретный сервер, можно было выбрать все серверы 'all', или к примеру, группу prod_group), -m (модуль, по сути в ансибл всё модули, тут просто проверка ping-pong)

![image](https://github.com/user-attachments/assets/a113f618-4402-4f8f-a9f8-fc58efebdbfb)

После указания hosts в ansible.cfg команда выглядит так ```ansible all -m ping```

![image](https://github.com/user-attachments/assets/cfaee11c-b562-48c4-bc88-bd2caf237f85)

```ansible all -m shell -a "uptime"``` ( -a аргумент, вместо 'shell' можно юзать 'command', но там не работают переменные,>,<,| итд)

![image](https://github.com/user-attachments/assets/9eb9b82e-fedd-4d03-9d2c-7d3f0a2bdf64)

```ansible linux1 -m setup``` инфа о серве

Можно создать файл 'dota.txt' ```ansible all -m file -a "path=/home/dota.txt state=touch" -b``` -b это sudo

либо скопировать созданный ```ansible all -m copy -a "src=dota.txt dest=/home/ mode=0777" -b``` 

и затем удалить его ```ansible all -m file -a "path=/home/hello state=absent" -b```

скачать что-то из инета ```ansible all -m get_url -a "url=https://dota3.ru dest=/home" -b```

проверить подключение ```ansible all -m uri -a "url=https://www.dota3.ru"``` и вывести его, дописав ```return_content=yes"```

установить ```ansible all -m apt -a "name=apache2 state=present" -b``` - ```state=absent``` удаляет

включить и добавить а автозагрузку```ansible all -m service -a "name=apache2 state=started enabled=yes" -b```

инфа о файле ```ansible all -m shell -a "ls /var" -v``` чем больше -vvvv, чем больше инфы
