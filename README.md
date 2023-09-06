# ROCKY VE DEBIAN DAĞITIMLARINDA WORDPRESS KURULUMU

Bu Dökümantasyon debian ve rocky dağıtımlarında wordpress kurulumunu açıklar.
Debian ve Rocky için ayrı ayrı iki farklı dökümantasyon vardır.

Öncelikle bulunduğunuz ortamda internet bağlantısı olduğuna, sanal makineyi kurduğunuz program üstünden network portlarını düzgün verdiğinize ve IP almada sorun yaşamadığınızdan emin olun.

# ROCKY

1. Kuruluma başlamadan önce wget, tar ve vim kurup, firewall servisinden 80 ve 443 portuna izin verilmesi gerekiyor.

* ``` dnf install wget ``` 
* ``` dnf install tar ``` 
* ``` dnf install vim ```
* ```systemctl status firewalld # servisin çalışıp çalışmadığını kontrol etmek için. ```
* Portları sırasıyla tcp protokolü ile açmak için
* - ```firewall-cmd --add-port=80/tcp --permanent```
* - ```firewall-cmd --add-port=443/tcp --permanent # https için şimdilik opsiyonel ``` 
* ```firewall-cmd --reload # servisi reload etmek için```
* ```firewall-cmd --list-all # açılan portları görmek için```
* ```systemctl status firewalld # servis durumunu görmek için ```

2.  Wordpress Kurulumu dağıtımı için bir web servis(apache), database(mysql) ve backend kodlarının bulunduran bir yazılım dili (php) kuracağız.
* ```dnf install mysql # mysql kurmak için```
* ```dnf install mysql-server # mysql-server kurmak için```
* - ```systemctl status mysqld # servisin durumunu görmek için```
* - ```systemctl enable --now mysqld # servisi çalıştırmak için```
* ```dnf install httpd # apache kurmak için ```
* - ```systemctl reload httpd # servisi yeniden başlatmak için```
* - ```systemctl status httpd # servis durumunu görmek için```
* Php kurabilmek için öncelikle güncel php_repolistini yüklemek zorundayız. [Güncel paketler her dağıtım için](https://rpms.remirepo.net/)
* - ```dnf install https://rpms.remirepo.net/enterprise/remi-release-9.rpm # Günümüz için```
* Kurduktan sonra aktif edilebilen modülleri listelemek için.
* - ```dnf module list php```
* - ```dnf module enable php:remi-8.3 # aktif edilebilen en güncel modül```
* mysql ile php bağlantısı için
* - ``` dnf install php-mysqlnd ```
* Son php paketini de kuruyoruz. [işlevi](https://www.php.net/manual/en/install.fpm.php)
* - ```dnf install php-fpm ```
* - ```systemctl enable --now php-fpm # çalıştırmak için```

3. DATABASE Konfigrasyonu
* ```mysql_secure_installation #  MySQL veritabanı sunucusunu kurduktan sonra güvenlik önlemleri almak için kullanılan kullanılır.  ```

> [!IMPORTANT]
> Burada root ve kullanıcı için şifre girmeye özen gösterin çünkü bazı sistemlerde root şifresiz olunca mysql açılmıyor.

- - ```CREATE DATABASE db_name; # Databse oluşturmak için ```
* User oluşturmak ve şifre ataması yapmak için burada da şifre vermeyi unutmayın.
- -  ```CREATE USER 'db_user'@'localhost' IDENTIFIED BY 'db_password';```
* Kullanıcıya ayrıcalık tanıyıp databaseyi kullanmasını sağlıyoruz.
- - ```GRANT ALL PRIVILEGES ON db_name.* TO 'db_user'@localhost; ```
* Ayrıcalıkları güncelliyoruz.
- - ```FLUSH PRIVILEGES;```
- - ``` EXIT; ```

4. Wordpressi wget kullanarak /var/www/ klasötrü içine kuruyoruz.
* ```cd /var/www/ ; wget https://wordpress.org/latest.tar.gz ```
* /var/www/ klasörü içine düzenli gözükmesi açısından sitemizin adında klasör ekliyoruz.
* - ```mkdir wp_first```;
* Arşiv dosyasını açıyoruz ve içindekileri wp_first klasörünün içine taşıyoruz.
* - ``` tar xf latest.tar.gz; mv ./wordpress/* ./wp_first/ ```
* Oluşturduğumuz klasöre ve içindeki dosyalara web servisi sahipliği atıyoruz (apache).
* - ```chown -R apache: ./wp_first/ ```
* /etc/httpd/conf.d/vhost_domain.com.conf konumundaki dosyanın içine aşağıdaki texti yazıyoruz.
```
<VirtualHost *:80>
    ServerName www.wp_first
    DocumentRoot /var/www/wp_first
    <Directory /var/www/wp_first>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```
* apachede sorun olup olmadığını kontrol etmek için
* - ```apachectl -t```
* /etc/hosts dosyası içine girip ip adresimize sitemizi yönlendiriyoruz.
* Son olarak aşağıdaki komutu kullanıp dosyanın içine girdikten sonra database bilgilerimizi tabloya ekliyince wordpress ile oluşan site çalışmaya başlayacak.
* - ``` mv /var/www/wp_first/wp-config-sample.php /var/www/wp_first/wp-config.php  ```
```
/** The name of the database for WordPress */
define( 'DB_NAME', 'db_name' );

/** Database username */
define( 'DB_USER', 'db_user' );

/** Database password */
define( 'DB_PASSWORD', 'db_password' );

/** Database hostname */
define( 'DB_HOST', 'localhost' );
```
* Servisleri yeniden başlatıyoruz ve sitemiz çalışmaya hazır.
```
   systemctl reload httpd
   systemctl reload php-fpm
   systemctl reload mysql-server
   systemctl reload mysqld
```

# DEBIAN

1. Kuruluma başlamadan önce wget, tar ve vim kurmanın yanı sıra firewall servisini de (ufw) kurup, firewall servisinden 80 ve 443 portuna izin verilmesi gerekiyor.

* ```apt install wget ``` 
* ```apt install tar ``` 
* ```apt install vim ```
* ```apt install ufw```
* Portları sırasıyla tcp protokolü ile açmak için
* - ```ufw allow 80/tcp```
* - ```ufw allow 443/tcp # https için şimdilik opsiyonel ```
* ```systemctl start ufw # servisi başlatmak için```
* ```ufw status # açılan portları görmek için```
* ```systemctl status ufw # servis durumunu görmek için ```
* ```ufw enable # servis statusunu aktif duruma getirmek için```

2.  Wordpress Kurulumu dağıtımı için bir web servis(apache), database(mariadb) ve backend kodlarının bulunduran bir yazılım dili (php) kuracağız.

* ```apt install mariadb-server # mariadb-server kurmak için```
* - ```systemctl status mariadb # servisin durumunu görmek için```
* - ```systemctl enable --now mariadb # servisi çalıştırmak için```
* ```apt install apache2 # apache kurmak için ```
* - ```systemctl reload apache2 # servisi yeniden başlatmak için```
* - ```systemctl status apache2 # servis durumunu görmek için```
* ```apt-get php php-mysql # php ve php ile mariadb arasındaki bağlantıyı sağlamak için  ```
* * Son php paketini de kuruyoruz. [işlevi](https://www.php.net/manual/en/install.fpm.php)
* - ```php --version # php versionumuza göre paketi kuracağız ```
* - ```apt install php8.2-fpm ```
* - ```systemctl start php8.2-fpm```
* - ```systemctl enable php8.2-fpm```
* - ```systemctl status php8.2-fpm```

3. DATABASE Konfigrasyonu
* ```mysql_secure_installation #  MySQL veritabanı sunucusunu kurduktan sonra güvenlik önlemleri almak için kullanılan kullanılır.  ```

> [!IMPORTANT]
> Burada root ve kullanıcı için şifre girmeye özen gösterin çünkü bazı sistemlerde root şifresiz olunca mysql açılmıyor.

- - ```CREATE DATABASE db_name; # Databse oluşturmak için ```
* User oluşturmak ve şifre ataması yapmak için burada da şifre vermeyi unutmayın.
- -  ```CREATE USER 'db_user'@'localhost' IDENTIFIED BY 'db_password';```
* Kullanıcıya ayrıcalık tanıyıp databaseyi kullanmasını sağlıyoruz.
- - ```GRANT ALL PRIVILEGES ON db_name.* TO 'db_user'@localhost; ```
* Ayrıcalıkları güncelliyoruz.
- - ```FLUSH PRIVILEGES;```
- - ``` EXIT; ```

4. Wordpressi wget kullanarak /var/www/ klasötrü içine kuruyoruz.
* ```cd /var/www/ ; wget https://wordpress.org/latest.tar.gz ```
* Arşiv dosyasını açıyoruz ve içindekileri html klasörünün içine taşıyoruz.
* - ```tar xf latest.tar.gz; mv ./wordpress/* ./html/ ```
* Oluşturduğumuz klasöre ve içindeki dosyalara web servisi sahipliği atıyoruz (apache).
* - ```chown -R www-data: ./html/ ```
* /etc/apache2/sites-available/wp_first.conf konumundaki dosyanın içine aşağıdaki texti yazıyoruz.
```
<VirtualHost *:80>
    ServerName www.wp_first.com
    DocumentRoot /var/www/wp_first
    <Directory /var/www/wp_first>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```
* apachede sorun olup olmadığını kontrol etmek için
* - ```apachectl -t```
* /etc/hosts dosyası içine girip ip adresimize sitemizi yönlendiriyoruz.
* Son olarak aşağıdaki komutu kullanıp dosyanın içine girdikten sonra database bilgilerimizi tabloya ekliyince wordpress ile oluşan site çalışmaya başlayacak.
* - ``` mv /var/www/html/wp-config-sample.php /var/www/html/wp-config.php  ```
```
/** The name of the database for WordPress */
define( 'DB_NAME', 'db_name' );

/** Database username */
define( 'DB_USER', 'db_user' );

/** Database password */
define( 'DB_PASSWORD', 'db_password' );

/** Database hostname */
define( 'DB_HOST', 'localhost' );
```
* Servisleri yeniden başlatıyoruz ve sitemiz çalışmaya hazır.
```
   systemctl reload apache2
   systemctl reload php8.2-fpm
   systemctl reload mariadb
```
