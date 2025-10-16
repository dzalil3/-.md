**alt-srv**
```tcl
mkdir /etc/net/ifaces/ens192
mkdir /etc/net/ifaces/ens224
mkdir /etc/net/ifaces/ens256
vim /etc/net/ifaces/ens192/options
    BOOTPROTO=dhcp
    TYPE=eth
vim /etc/net/ifaces/ens224/options
    BOOTPROTO=static
    CONFIG_IPV4=yes
    DISABLED=no
    TYPE=eth
cp /etc/net/ifaces/ens224/options /etc/net/ifaces/ens256/options
vim /etc/net/ifaces/ens224/ipv4address
    192.168.1.10/24
vim /etc/net/ifaces/ens256/ipv4address
    192.168.2.10/24
vim /etc/net/sysctl.conf
    net.ipv4.forward=1
systemctl restart network
apt-get update && apt-get install iptables
iptables -t nat -A POSTROUTING -o ens192 -s 0/0 -j MASQUERADE
iptables-save > /etc/sysconfig/iptables
```
**win-srv**
>[ip address: 192.168.1.1]
>[mask: 255.255.255.0]
>[gateway: 192.168.1.10]
>[dns1: 8.8.8.8]
>[dns2: 1.1.1.1]
###Выключить firewall(брандмауэр) в панели управления.
**CLI**
>[ip address: 192.168.2.1]
>[mask: 255.255.255.0]
>[gateway: 192.168.2.10]
>[dns1: 8.8.8.8]
>[dns2: 1.1.1.1]
###Выключить firewall(брандмауэр) в панели управления.

**alt-srv**

```tcl
apt-get install nginx php8.2-fpm-fcgi
systemctl start nginx php8.2-fpm
systemctl enable nginx php8.2-fpm
vim /etc/nginx/nginx.conf
```
###Удалить_всё_содержимое
```tcl
events {
    worker_connections 1024;
}

http {
    server {
        listen 80;
        server_name localhost;
        root /var/www/html;
        index index.php index.html;

        location / {
            try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
            fastcgi_pass unix:/var/run/php8.2-fpm/php8.2-fpm.sock;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
        }
    }
}
```
```tcl
vim /var/www/html/index.php
```
```tcl
<?php
$file = 'tickets.txt';

if ($_POST['subject']) {
    $ticket = [
        'id' => time(),
        'subject' => $_POST['subject'],
        'author' => $_POST['author'],
        'status' => 'Новая',
        'date' => date('d.m.Y H:i'),
        'description' => $_POST['description']
    ];
    
    file_put_contents($file, json_encode($ticket)."\n", FILE_APPEND);
    echo "<div style='background:green; color:white; padding:10px;'>Заявка создана!</div>";
}
?>
<!DOCTYPE html>
<html>
<head>
    <title>Система учёта заявок</title>
    <style>
        body { font-family: Arial; margin: 20px; }
        .ticket { border: 1px solid #ccc; margin: 10px 0; padding: 15px; }
        .new { background: #e8f4fd; }
        form { background: #f5f5f5; padding: 20px; }
    </style>
</head>
<body>
    <h1>📋 Система учёта заявок</h1>
    <form method="post">
        <h3>Создать новую заявку:</h3>
        <input type="text" name="author" placeholder="Ваше имя" required><br><br>
        <input type="text" name="subject" placeholder="Тема заявки" required><br><br>
        <textarea name="description" placeholder="Описание проблемы" rows="4" style="width:300px;"></textarea><br><br>
        <button type="submit">Создать заявку</button>
    </form>
    <hr>
    <h2>Список заявок:</h2>
    <?php
    if (file_exists($file)) {
        $tickets = file($file);
        foreach (array_reverse($tickets) as $line) {
            $ticket = json_decode($line, true);
            echo "<div class='ticket new'>";
            echo "<strong>Заявка #{$ticket['id']}</strong><br>";
            echo "<strong>Тема:</strong> {$ticket['subject']}<br>";
            echo "<strong>Автор:</strong> {$ticket['author']}<br>";
            echo "<strong>Статус:</strong> {$ticket['status']}<br>";
            echo "<strong>Дата:</strong> {$ticket['date']}<br>";
            echo "<strong>Описание:</strong> {$ticket['description']}<br>";
            echo "</div>";
        }
    } else {
        echo "<p>Заявок пока нет</p>";
    }
    ?>
</body>
</html>
```
```tcl
systemctl restart nginx
systemctl stop httpd2.service
systemctl disable httpd2.service

```
>[Проверка: nginx -t,curl http://localhost. На cli рткрыть edge и перейти по адрессу http://192.168.1.10]

**АЛЬТЕРНАТИВА**

**alt-srv**
```tcl
iptables -t nat -A POSTROUTING -o ens192 -s 0/0 -j MASQUERADE
iptables -I INPUT -p tcp --dport 80 -j ACCEPT
iptables -I INPUT -p tcp --dport 8000 -j ACCEPT
iptables-save > /etc/sysconfig/iptables
systemctl enable --now iptables
```
```tcl
apt-get install nginx php8.2-fpm-fcgi
vim /etc/nginx/nginx.conf
```
###Удалить всё содержимое и вставить следующий кнфиг:
```tcl
user nobody;
worker_processes auto;

events {
    worker_connections 1024;
}

http {
    proxy_temp_path /var/spool/nginx/tmp/proxy;
    fastcgi_temp_path /var/spool/nginx/tmp/fastcgi;
    client_body_temp_path /var/spool/nginx/tmp/client;
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    sendfile on;
    gzip on;
    gzip_types text/plain text/css text/xml application/x-javascript application/atom+xml;

    server {
        listen 80;
        server_name localhost;
        root /var/www/html;
        index index.php index.html;

        location / {
            try_files $uri $uri/ =404;
            allow all;
        }

        location ~ \.php$ {
            fastcgi_pass unix:/var/run/php8.2-fpm/php8.2-fpm.sock;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
            allow all;
        }
    }
}
```
```tcl
vim /var/www/html/create.php
```
```tcl
<?php
$file = 'tickets.txt';

if ($_POST['subject']) {
    $ticket = [
        'id' => time(),
        'subject' => $_POST['subject'],
        'author' => $_POST['author'],
        'status' => 'Новая',
        'date' => date('d.m.Y H:i'),
        'description' => $_POST['description']
    ];
    
    file_put_contents($file, json_encode($ticket)."\n", FILE_APPEND);
    echo "Заявка создана!";
}
?>
<!DOCTYPE html>
<html>
<head><title>Создание заявки</title></head>
<body>
    <h1>Создание заявки</h1>
    <form method="post">
        <input type="text" name="author" placeholder="Ваше имя" required><br>
        <input type="text" name="subject" placeholder="Тема заявки" required><br>
        <textarea name="description" placeholder="Описание" required></textarea><br>
        <button type="submit">Создать</button>
    </form>
</body>
</html>
```
```tcl
vim /var/www/html/view.php
```
```tcl
<?php
$file = 'tickets.txt';
?>
<!DOCTYPE html>
<html>
<head><title>Просмотр заявок</title></head>
<body>
    <h1>Все заявки</h1>
    <?php
    if (file_exists($file)) {
        $tickets = file($file);
        foreach (array_reverse($tickets) as $line) {
            $ticket = json_decode($line, true);
            echo "<div style='border:1px solid #ccc; margin:10px; padding:10px;'>";
            echo "<strong>Заявка #{$ticket['id']}</strong><br>";
            echo "<strong>Тема:</strong> {$ticket['subject']}<br>";
            echo "<strong>Автор:</strong> {$ticket['author']}<br>";
            echo "<strong>Статус:</strong> {$ticket['status']}<br>";
            echo "<strong>Дата:</strong> {$ticket['date']}<br>";
            echo "<strong>Описание:</strong> {$ticket['description']}<br>";
            echo "</div>";
        }
    } else {
        echo "<p>Заявок нет</p>";
    }
    ?>
</body>
</html>
```
```tcl
chown -R nobody:nobody /var/www/html/
chmod 755 /var/www/html/
chmod 644 /var/www/html/*.php
chmod 666 /var/www/html/tickets.txt
```

