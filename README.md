# Инструкция по базовой настройке сервера
В этой инструкции описывается стандартная процедура настройки любого сервера, вне зависимости от его предназначения.

## Настройка подключения по SSH
1. Создать нового пользователя `www`
```
adduser www
```
2. Добавить пользователя `www` в группу `sudo` для того, чтобы он мог выполнять инструкции с правами суперпользователя
```
usermod -aG sudo www
```
3. Теперь можно подключиться по SSH от имени пользователя `www` и все дальнейшие операции производить от него. Но при этому, от `root` пока что лучше не отключаться, чтобы в случае необходимости была возможность что-то исправить.
4. Скопировать SSH-ключи на сервер (команда выполняется **на локальном компьютере**)
```
ssh-copy-id www@<IP-address>
```
После этого можно будет подключаться к серверу без ввода пароля
```
ssh www@<IP-address>
```
5. На **локальном компьютере** можно создать файл `.ssh/config` и прописать в него следующее (заменив <IP-address> на реальный IP-адрес сервера, а NameForYou - на удобное имя):
```
Host NameForYou
    HostName <IP-address>
    IdentityFile ~/.ssh/id_rsa
    User www
```
После этого можно будет подключаться к серверу командой
```
ssh NameForYou
```
6. Отключить возможность SSH-авторизации через пароль и под пользователем `root`. Для этого в файле `/etc/ssh/sshd_config` необходимо изменить значения переменных
```
AllowUsers www
PermitRootLogin no
PasswordAuthentication no
```
7. Перезагрузить SSH-сервис
```
sudo service ssh restart
```

## Базовая настройка
1. Установить часовой пояс
```
sudo dpkg-reconfigure tzdata
```
2. Изменить имени сервера
```
sudo hostnamectl set-hostname <New Name>
```
3. Установить и настроить брандмаузер `ufw`
```bash
sudo apt install ufw
```
Блокируем все входящие соединения, разрешая только подключение по SSH.
```bash
sudo ufw default deny incoming; \
sudo ufw default allow outgoing; \
sudo ufw allow ssh; \
sudo ufw enable
```
4. Установить и настроить `fial2ban` для блокирования IP-адресов, пытающихся подобрать пароль к SSH.
```bash
sudo apt install fail2ban && sudo systemctl enable fail2ban && sudo systemctl start fail2ban
```
Отредактировать настройки `fail2ban` в файле `/etc/fail2ban/jail.local`
```
[DEFAULT]
bantime = 604800
maxretry = 3
findtime = 7200
banaction = ufw
[sshd]
enabled = true
```
`bantime` - время в секундах, на которое банится IP-адрес  
`maxretry` - количество возможных попыток до попадания в бан-список  
`findtime` - время, в течение которого считаются неудачные попытки  
`banaction` - брандмаузер, который будет блокировать iP-адреса  
  
Посмотреть статус можно командой
```bash
sudo fail2ban-client status sshd
```

## Установка базового набора программ
1. Установить минимальный набор необходимых программ
```
sudo apt-get update ; \
sudo apt-get install -y vim tmux htop git curl wget unzip zip zsh tree;
```

## Установка oh-my-zsh и плагинов
1. Установить `oh-my-zsh`
```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
2. Установить плагины для автодополнения и подсветки синтаксиса
```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```
```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```
3. Прописать установленные плагины в файл `~/.zshrc` в секцию `plugins`:
```
plugins=( 
    # other plugins...
    zsh-autosuggestions
    zsh-syntax-highlighting
)
```
4. Изменить тему оформления в файле `~/.zshrc` (например, на `gnzh`)
5. Перезагрузить настройки ZSH
```
source ~/.zshrc
```

## Настройка tmux
Подробная инструкция по настройке `tmux` находится в репозитории [cheatsheet-tmux](https://github.com/Shecspi/cheatsheet-tmux)

## Настройка nVim
Подробная инструкция по настройке `tmux` находится в репозитории [cheatsheet-nvim](https://github.com/Shecspi/cheatsheet-nvim)
