# Шарим (ShareIt?)

1. Установите пакет samba
2. Что такое общая папка, зачем оно может быть нужно?
3. Создайте общую папку без пароля с правами только на чтение файлов
4. Создайте общую папку с паролем с правами на чтение и запись
5. Создайте общую папку с доступом для какой-то группы с полными правами
6. Создайте общую папку в которой у одной группы будет полный доступ, а у другой только доступ на чтение.
Третья группа не должна иметь к ней доступа

Ответ:

1. Установите пакет samba
apt-get install samba

2. Что такое общая папка, зачем оно может быть нужно?
Эти папки могут использоваться разными пользователями в том числе с других ОС.

3. Создайте общую папку без пароля с правами только на чтение файлов
mkdir -p /srv/samba/public
chmod 777 /srv/samba/public
chown nobody:nogroup /srv/samba/public

В данном конфиге:
sudo vim /etc/samba/smb.conf

Дописываем в конец:
[public]
   path = /srv/samba/public
   guest ok = yes
   read only = yes
   browseable = yes

4. Создайте общую папку с паролем с правами на чтение и запись
mkdir -p /srv/samba/secured
chmod 770 /srv/samba/secured

Настроим нового пользователя:
useradd kakoytouser
smbpasswd -a 1234
chown kakoytouser:kakoytouser /srv/samba/secured

В данном конфиге:
sudo vim /etc/samba/smb.conf

Дописываем в конец:
[secured]
path = /srv/samba/secured
browseable = yes
writable = yes
valid users = kakoytouser
guest ok = no

5. Создайте общую папку с доступом для какой-то группы с полными правами
mkdir -p /srv/samba/group
groupadd sambagroup
chown :sambagroup /srv/samba/group
chmod 770 /srv/samba/group

В данном конфиге:
sudo vim /etc/samba/smb.conf

Дописываем в конец:
[group]
path = /srv/samba/group
read only = no
valid users = @sambagroup

По аналогии с добавлением в группу суперпользователя:
usermod -aG sambagroup kakoytouser

6. Создайте общую папку в которой у одной группы будет полный доступ, а у другой только доступ на чтение.
Третья группа не должна иметь к ней доступа
groupadd read_access
mkdir -p /srv/samba/mixed
chmod 770 /srv/samba/group
chown root:sambagroup /srv/samba/group

В данном конфиге:
sudo vim /etc/samba/smb.conf

Дописываем в конец:
[group]
path = /srv/samba/group
browseable = yes
writable = no
valid users = @sambagroup, @read_access
write list = @sambagroup
force group = sambagroup
create mask = 0660
directory mask = 0770

Добавляем нескольких пользователей и добавляем их в группы с разными правами доступа:
useradd -m user1
useradd -m user2
usermod -aG sambagroup user1
usermod -aG read_access user2
smbpasswd -a user1
smbpasswd -a user2
