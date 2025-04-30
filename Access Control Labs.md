#### 1.Lab: Unprotected admin functionality

Korunaksız bir admin paneli bulunmaktadır.
Adımlar:
- robots.txt iste
- gizlenmiş administrator-panel sayfasını iste
- Carlos kullanıcısını sil

#### 2.Lab: Unprotected admin functionality with unpredictable URL

Labda gizli bir yerde admin panelin olduğu söyleniyor
- Sayfa kaynağına batığımızda toplinks section'ın altında bir scriptte admin panelinin nerede oluğu yazmakta
- URL'e /admin-6zepfbekliyoruz
- carlos'u siliyoruz

#### 3.Lab: User role controlled by request parameter

- Labda Admin=false şeklinde bir cookie var
-  bu değeri true olarak değiştiriyoruz
- GET /admin/delete?username=carlos isteğini atıyoruz

#### 4.Lab: User role can be modified in user profile

Bu laboratuvarda `/admin` yolunda bir yönetici paneli bulunmaktadır. Bu panele yalnızca **roleid değeri 2 olan** (yani yönetici yetkisine sahip) **giriş yapmış kullanıcılar** erişebilir.
Laboratuvarı çözmek için:
- **Yönetici paneline erişin**
- Ardından **carlos adlı kullanıcıyı silin**
Kendi hesabınızla giriş yapmak için şu kimlik bilgilerini kullanabilirsiniz:
**Kullanıcı adı:** `wiener`  
**Şifre:** `peter`

Adımlar:
- giriş yaptıkatan sonra email adresi değiştiriyoruz
- cevap olarak bir JSON objesi dönüyor
- bu nesnede roleid:1 şeklinde bir ibare görüyoruz
- tekrar mail değiştirme isteğini yolluyoruz ama bu sefer roleid değişkeni ekleyip değerini 2 yapıyoruz
- Browserdan admin paneline girip carlosu siliyoruz


#### 5.Lab: User ID controlled by request parameter

Bu laboratuvarda, kullanıcı hesap sayfasında **yatay yetki yükseltme (horizontal privilege escalation)** zafiyeti bulunmaktadır.
Laboratuvarı çözmek için:  
**carlos adlı kullanıcının API anahtarını (API key)** ele geçirin ve bunu çözüm olarak gönderin.
Kendi hesabınızla giriş yapmak için şu kimlik bilgilerini kullanabilirsiniz:
**Kullanıcı adı:** `wiener`  
**Şifre:** `peter`

Adımlar:
- Hesaba giriş yapıyoruz
- GET /my-account?id=wiener isteğindeki id değerini carlos yapıyoruz
- sayfada carlosun API key değeri bulunmakta:
- 9iQsoRtZZMSH8b8Nkr5Cd4oXnU8vCCDD


#### 6.Lab: User ID controlled by request parameter, with unpredictable user IDs

Bu laboratuvarda, kullanıcı hesap sayfasında **yatay yetki yükseltme (horizontal privilege escalation)** zafiyeti bulunmaktadır, ancak kullanıcılar **GUID** ile tanımlanmaktadır.
Laboratuvarı çözmek için:  
**carlos adlı kullanıcının GUID değerini bulun** ve ardından onun **API anahtarını (API key)** ele geçirip çözüm olarak gönderin.
Kendi hesabınızla giriş yapmak için şu kimlik bilgilerini kullanabilirsiniz:
**Kullanıcı adı:** `wiener`  
**Şifre:** `peter`

Adımlar:
- Carlos tarafından yazılan bir post buluyoruz
- URL de GUID si var
- Pu7PsV5mtXPZCGNDZxDWEyVVrip1Mq8O

#### 7.Lab: User ID controlled by request parameter with data leakage in redirect

This lab contains an access control vulnerability where sensitive information is leaked in the body of a redirect response.

Adımlar:
- Girş yapıyoruz
- GET /my-account?id=wiener isteğinde id=carlos yazıyoruz
- sayfada carlos'un API anahtarı var
- XdsjcKonYzM0jJIM1C05jimzqy5ftpAD

#### 8.Lab: User ID controlled by request parameter with password disclosure

This lab has user account page that contains the current user's existing password, prefilled in a masked input.

Adımlar:
- GET /my-account?id=administrator isteğini atıyoruz
- sayfa kaynağından masked input değerini buluyoruz
- administrator olarak giriş yapıyoruz
- carlos kullanıcısını siliyoruz

#### 9.Lab: Insecure direct object references

This lab stores user chat logs directly on the server's file system, and retrieves them using static URLs.
Solve the lab by finding the password for the user `carlos`, and logging into their account.

Adımlar:
- Chat kısmına gir view transcript butonuna bas
- GET /download-transcript/2.txt isteğinin gönderildiğini incele
- GET /download-transcript/1.txt isteğini gönder
- dosya da şifreyi bul
- caros olarak giriş yap

#### 10.Lab: URL-based access control can be circumvented

NOTE:
This rule denies access to the `POST` method on the URL `/admin/deleteUser`, for users in the managers group. Various things can go wrong in this situation, leading to access control bypasses.

Some application frameworks support various non-standard HTTP headers that can be used to override the URL in the original request, such as `X-Original-URL` and `X-Rewrite-URL`. If a website uses rigorous front-end controls to restrict access based on the URL, but the application allows the URL to be overridden via a request header, then it might be possible to bypass the access controls using a request like the following:

`POST / HTTP/1.1 X-Original-URL: /admin/deleteUser ...`

Bu web sitesinde, `/admin` yolunda **kimlik doğrulama gerektirmeyen bir yönetici paneli** bulunmaktadır. Ancak, **ön yüz (front-end) sistemi**, dış erişimi bu yola engelleyecek şekilde yapılandırılmıştır.

Bununla birlikte, arka uç (back-end) uygulaması, **`X-Original-URL` başlığını (header)** destekleyen bir framework üzerine inşa edilmiştir

Adımlar:
- /admin dene access denied mesajı gelsin
- bu sefer home sayfasını istereken header olarak X-Original-URL: /admin ekle
- görüceksin ki admin sayfasına erişebildin
- GET /?username=carlos HTTP/2 isteğini X-Original-URL: /admin/delete ile gönder
- Carlos kullanıcısı silinmiş olacak

#### 11.Lab: Method-based access control can be circumvented

Bu labda istek metotlarının bloke edildiği durumda nasıl access control vulnerabity bulup sömüreceğimizi test ediyor

Adımlar:
- labda verilen credentials ile admin olarak giriş yapıyoruz
- carlos kullanıcısının yetkisini yükseltiyoruz
- bu isteği repeater'a gönderiyoruz `POST /admin-roles HTTP/2`
- kendi hesabımızdan bir istek gönderiyoruz ve bu istekteki sessionid cookie değerini  kopyalıyoruz
- bu değeri yetki yükseltme isteğine koyuyoruz
- username değerini wiener olarak değiştiriyoruz
- İstek gönderildiğinde 'Unauthorized' uyarısını görüyoruz
- İsteği POST'tan GET isteğine çevirip gönderiyoruz
- yetkimiz yükselmiş oluyor.

#### 12.Lab: Multi-step process with no access control on one step

NOTE: Sometimes, a website will implement rigorous access controls over some of these steps, but ignore others. Imagine a website where access controls are correctly applied to the first and second steps, but not to the third step. The website assumes that a user will only reach step 3 if they have already completed the first steps, which are properly controlled. An attacker can gain unauthorized access to the function by skipping the first two steps and directly submitting the request for the third step with the required parameters.

This lab has an admin panel with a flawed multi-step process for changing a user's role. You can familiarize yourself with the admin panel by logging in using the credentials `administrator:admin`.
To solve the lab, log in using the credentials `wiener:peter` and exploit the flawed access controls to promote yourself to become an administrator.

Adımlar:
- Log in as admin to copy a request
	- The one POST /admin-roles request with the data :  action=upgrade&confirmed=true&username=wiener
- Access control has been applied on the firs request but this request doesn't have access control
- Now log in as wiener and copy this user's sessionid
- paste it as the new sessionid in the request mentioned and send it
- the user wiener now is and admin

#### 13.Lab: Referer-based access control

Bazı sayfalar erişim kontrolü sağlarken sadece referans başlığına dikkat eder. Bu labda buna örnek bir uygulama var

Steps:
- log in as administrator
- send a request to upgrade carlos
- send the request to burp repeater
- log in as wiener
- paste your sessionId on the previously mentioned request
- change the username to wiener
- send it now you are an admin
