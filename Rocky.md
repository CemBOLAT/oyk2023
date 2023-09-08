
# ROCKY

1. Kuruluma başlamadan önce wget, tar ve vim kurup, firewall servisinden 80 ve 443(opsiyonal) portuna izin verilmesi gerekiyor.

## Paketlerin kurulumu
<p align="center"> Paketler inerken aşağıdaki gibi bir ekran gelecek direkt 'y' tuşuna basarsınız paket inmeye başlayacak. </p>
<p align="center">
<img width="300"  alt="Ekran Resmi 2023-09-08 13 07 42" src="https://github.com/CemBOLAT/oyk2023/assets/103999323/a0b44376-4027-4100-8118-1e99289a8a3c">
</p>
<p align="center"> Paketiniz zaten yüklü ise aşağıdaki gibi bir ekran karşınıza çıkacak </p>
<p align="center">
<img width="300" alt="Ekran Resmi 2023-09-08 14 13 47" src="https://github.com/CemBOLAT/oyk2023/assets/103999323/6f03c91b-4f6d-4a34-87e5-bb0ff6244c4d">
</p>

* * ```
    dnf install wget
* * ```
    dnf install tar
* * ```
    dnf install vim
* Firewall servisinin durumunu görüntülemek için.
* - ```
    systemctl status firewalld
* Portları sırasıyla tcp protokolü ile açmak için
* - ```
    firewall-cmd --add-port=80/tcp --permanent
* - Opsiyonel (https)
* - ```
    firewall-cmd --add-port=443/tcp --permanent
> [!Note]
> Servisler için reload komutu yapılan değişikliklerin servisi yeniden başlatmak yerine servisin yapılandırma dosyalarını yeniden yükler ve günceller.
* Servisin yapılandırma dosyalarını güncellemek için
* * ```
    firewall-cmd --reload
* Açılan portları görebilmek için
* * ```
    firewall-cmd --list-all
* Firewall servisinin durumunu görüntülemek için.
* * ```
    systemctl status firewalld
2.  Wordpress Kurulumu dağıtımı için bir web servis(apache), database(mysql) ve backend kodlarının bulunduran bir yazılım dili (php) kuracağız.
* MYSQL kurmak için
* * ```
    dnf install mysql
* MYSQL-Server kurmak için
* * ```
    dnf install mysql-server
* Servis durumunu görüntülemek için
* - ```
    systemctl status mysqld
* Servisi sistem açılırken açmak ve şuan çalıştırmak için
* - ```
    systemctl enable --now mysqld
* Apache kurmak için
* * ```
    dnf install httpd
* Php kurabilmek için öncelikle güncel php_repolistini yüklemek zorundayız. [Güncel paketler her dağıtım için](https://rpms.remirepo.net/)
* - ```
    dnf install https://rpms.remirepo.net/enterprise/remi-release-9.rpm
* Kurduktan sonra aktif edilebilen modülleri listelemek için.
* - ```
    dnf module list php
### Aktif edilebilen modüllerin görüntüsü
* php-8.3'ün yanında "[e]" yazmasını sebebi altta aktifleştirmiş olmam
<p>
<img width="200" alt="Ekran Resmi 2023-09-08 14 17 49" src="https://github.com/CemBOLAT/oyk2023/assets/103999323/0e3a00e2-740e-426f-a535-25bad49dd939">
</p>

* Modülü aktifleştitmek için
* - ```
    dnf module enable php:remi-8.3
* mysql ile php bağlantısı için
* - ```
    dnf install php-mysqlnd
* Son php paketini de kuruyoruz. [işlevi](https://www.php.net/manual/en/install.fpm.php)
* - ```
    dnf install php-fpm
* Servisi sistem açılırken açmak ve şuan çalıştırmak için
* - ```
    systemctl enable --now php-fpm
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
    chown -R apache: ./wp_first/
* - Bulunduğumuz klasördeki dosyaları listeliyoruz ve .tar.gz uzantılı dosya ile wordpress klasörünü siliyoruz.
* - ```
    ls
    rm -rf ./wordpress <indirdiğimiz dosya adı;
* /etc/httpd/conf.d/wp_first.conf konumundaki dosyanın içine aşağıdaki texti yazıyoruz.
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
* - ```
    apachectl -t
* * Hatasız olduğunda syntax on yazması gerekir eper syntax hatası var ise hatanın olduğu dosya/dosyaları kontrol edin.
    <img width="100" alt="Ekran Resmi 2023-09-08 14 24 49" src="https://github.com/CemBOLAT/oyk2023/assets/103999323/7984b812-0baf-4b23-8e72-09cfd61dd6cc">
* /etc/hosts dosyası içine girip ip adresimizi sitemize yönlendiriyoruz.

* Son olarak aşağıdaki komutu kullanıp dosyanın içine girdikten sonra database bilgilerimizi tabloya ekliyince wordpress ile oluşan site çalışmaya başlayacak.
* - ```
    mv /var/www/wp_first/wp-config-sample.php /var/www/wp_first/wp-config.php
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
* Servislerin yapılandırma dosyalarını tarayıp güncelliyoruz veya yeniden başlatıyoruz. 
```
   systemctl reload httpd
   systemctl reload php-fpm
   systemctl restart mysqld
```
* Son olarak ip a yazıyoruz ve 127.0.0.1 dışındaki ipv4 adresimizi kopyalayıp tarayıcımıza yapıştırınca sitemiz açılacak
