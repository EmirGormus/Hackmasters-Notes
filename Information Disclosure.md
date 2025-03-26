#### 1.Lab: Information disclosure in error messages

Error message almak için `product?productId=4e` gibi bir sorgu giriyoruz zira uygulama bu sorguyu yaparken veri tipi uyuşmadığı için hata veriyor, verdiği hata mesajında da kullandığı framework'ün sürümünü veriyor `Apache Struts 2 2.3.31`.

#### 2.Lab: Information disclosure on debug page

`GET /product?productId=2 HTTP/2` isteğini attığımızda gelen html dosyasının içinde bir yorum ifadesi görüyoruz `<!-- <a href=/cgi-bin/phpinfo.php>Debug</a> -->`. Belirtilen dosyaya istek gönderdiğimizde `GET /cgi-bin/phpinfo.php HTTP/2` SECRET_Key değerini bulabiliyoruz.

ekcgvcj5tj8f0mniuhegtt54obiibgp5

#### 3.Lab: Source code disclosure via backup files

**Robots.txt**: Bu dosya arama motorlarına hangi sayfaların görünmesine ve görünmemesine dair yönrege verir.

robots.txt dosyasını istiyoruz. 
```
User-agent: *
Disallow: /backup```
```
içinde /backup adlı bir dosyanın izinsiz olduğu gösterir. Bu dosyaya navige ettiğimizde backup bir dosya görüyoruz onun içinde de database şifresini buluyoruz

![[Pasted image 20250326014329.png]]

şifre :g7se8i2nv5m0wmfa922m5rz4wje04gjz

#### 4.Lab: Authentication bypass via information disclosure

Bu labda carlos adlı kullanıcıyı silmemiz gerekiyor
TRACE metodu kullanıyoruz
TRACE /admin isteği atıyoruz
gelen cevapta 
X-Custom-Ip-Authorization: 127.0.0.1 gibi bir ibare görüyoruz.
bu ibareyi eklemek bizi local host gibi gösteriyor. header ile admin paneline ulaşıyoruz. admin paneli üzerinden carlosu siliyoruz.

#### 5.Lab: Information disclosure in version control history

Bu labda .git dizinine gidiyoruz burda önceki sürümlerde  bir şey var mı diye bakacağız. bunun için repoyu indiriyoruz:
wget -r https://0a8e009c046ea32e81d3163c007400c2.web-security-academy.net/.git/

Ardından da bu repoyu git GUI ile açıyoruz.  burada commit history'ye baktığımızda commitlerden birinde admin.conf dosyasında şifrenin yazılı olduğunu görüyoruz.

`+ADMIN_PASSWORD=v9oiw6xuny68jsh7mobr`

Bu şifreyi administrator olarak giriyoruz ve carlos kullanıcısını siliyoruz.