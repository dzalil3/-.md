alt-srv
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
```
