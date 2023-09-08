
# DEBIAN

1. Kuruluma başlamadan önce wget, tar ve vim kurmanın yanı sıra firewall servisini de (ufw) kurup, firewall servisinden 80 ve 443 portuna(opsiyonel) izin verilmesi gerekiyor.

## Paketlerin kurulumu
<p align="center"> Paketiniz zaten yüklü ise aşağıdaki gibi bir ekran karşınıza çıkacak </p>
<p align="center">
<img width="300" alt="Ekran Resmi 2023-09-08 15 04 58" src="https://github.com/CemBOLAT/oyk2023/assets/103999323/ee442a8f-9647-46f7-b504-a4f241bfac01">
</p>

* ```
  apt install wget
* ```
  apt install tar
* ```
  apt install vim
* ```
  apt install ufw
* Portları sırasıyla tcp protokolü ile açmak için
* - ```
    ufw allow 80/tcp
* - ```
    ufw allow 443/tcp
* Ufw servisinin durumunu görüntülemek için.
* * ```
    systemctl status ufw # servis durumunu görmek için
* Servisi başlatmak için 
* * ```
    systemctl start ufw
* Açılan portları görmek için
* * ```
    ufw status
* Servisi bilgisayar açıldığında açılan servislere eklemek için
* * ```
    ufw enable
2.  Wordpress Kurulumu dağıtımı için bir web servis(apache), database(mariadb) ve backend kodlarının bulunduran bir yazılım dili (php) kuracağız.
* Mariadb-server kurmak için
* * ```
    apt install mariadb-server
* Servisin durumunu görmek için
*  * ```
      systemctl status mariadb
* Servisi sistem açılırken açmak ve şuan çalıştırmak için
*  * ```
      systemctl enable --now mariadb
* Apache kurmak için
* * ```
    apt install apache2
* Servisin yapılandırma dosyalarını güncellemek için
* -  ```
      systemctl reload apache2
* Servisin durumunu görmek için
* * ```
    systemctl status apache2
* php ve php ile mariadb arasındaki bağlantıyı sağlamak için
* * ```
    apt-get php php-mysql
* * Son php paketini de kuruyoruz. [işlevi](https://www.php.net/manual/en/install.fpm.php)
* php versionumuza göre paketi kuracağız
* - ```
    php --version
<p align="center">
  <img width="300" alt="Ekran Resmi 2023-09-08 15 04 58" src="https://github.com/CemBOLAT/oyk2023/assets/103999323/fdc39d90-94e0-4947-a7cd-d06e61634acb">
</p>

* - ```
    apt install php8.2-fpm
* - ```
    systemctl start php8.2-fpm
* - ```
    systemctl enable php8.2-fpm
* - ```
    systemctl status php8.2-fpm
3. DATABASE Konfigrasyonu
*  MySQL veritabanı sunucusunu kurduktan sonra güvenlik önlemleri almak için kullanılan kullanılır.
* * ```
    mysql_secure_installation
> [!IMPORTANT]
> Burada root ve kullanıcı için parola atamaya özen gösterin çünkü bazı sistemlerde root parolasız olunca mysql açılmıyor.
* Database oluşturmak için
- - ```
    CREATE DATABASE db_name;
* User oluşturmak ve şifre ataması yapıyoruz. Burada da şifre vermeyi unutmayın.
- - ```
    CREATE USER 'db_user'@'localhost' IDENTIFIED BY 'db_password';
* Kullanıcıya ayrıcalık tanıyıp databaseyi kullanmasını sağlıyoruz.
- - ```
    GRANT ALL PRIVILEGES ON db_name.* TO 'db_user'@localhost;
* Ayrıcalıkları güncelliyoruz.
- - ```
    FLUSH PRIVILEGES;
- - ```
    EXIT;
4. Wordpressi wget kullanarak /var/www/ klasötrü içine kuruyoruz.
* ```
  cd /var/www/ ; wget https://wordpress.org/latest.tar.gz
* /var/www/ klasörü içine düzenli gözükmesi açısından sitemizin adında klasör ekliyoruz.
* - ```
    mkdir wp_first;
* Arşiv dosyasını açıyoruz ve içindekileri wp_first klasörünün içine taşıyoruz.
* - ```
    tar xf latest.tar.gz; mv ./wordpress/* ./wp_first/
* Oluşturduğumuz klasöre ve içindeki dosyalara web servisi sahipliği atıyoruz (apache).
* - ```
    chown -R www-data: ./wp_first/
* - Bulunduğumuz klasördeki dosyaları listeliyoruz ve .tar.gz uzantılı dosya ile wordpress klasörünü siliyoruz.
* - ```
    ls
    rm -rf ./wordpress <indirdiğimiz dosya adı;
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
