#### [1.Lab: JWT authentication bypass via unverified signature](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-unverified-signature)

In this Lab it is noted that the application doesn't verify the signature which means if just change the creadentials we are sending it should give us access to certain unauthorized parts of the application

Steps:
- Login as `wiener:peter`
- Send a request for the `/admin` endpoint
- You will see that you are unauhtorized
- Use the JWT editor to change the credentials of the JWT
- Change the `wiener` to `administrator`
- Send the request with an empty signature for good measure
- See that you are now able to reach the endpoint
- Now send a `POST /admin/delete?username=carlos HTTP/2` with the same JWT
- Voila, Lab solved !

#### [2.Lab: JWT authentication bypass via flawed signature verification](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-flawed-signature-verification)

We apply the same steps as the previous lab with one difference, we set the `alg` to `none` because the application doesn't verify the stuff we write based on an algorithm it has dedicated but on the one we give it. Lab solved

#### 3.Lab: JWT authentication bypass via weak signing key

Adımlar
- Hashcat ile secret'i bul: 
```
hashcat -a 0 -m 16500 eyJraWQiOiJjMjFmNmI2MS03NzYxLTQ0YTYtOTE4Ny1jZDQ2ODM2MzY4YTIiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc0NzQ0MzU4NSwic3ViIjoid2llbmVyIn0.kQ8-DWBnsHlmIvEvcYDZmBVzpuJ89CeTa4A6zx71n8M jwt.secrets.list
```

- Sonuç:
```
eyJraWQiOiJjMjFmNmI2MS03NzYxLTQ0YTYtOTE4Ny1jZDQ2ODM2MzY4YTIiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc0NzQ0MzU4NSwic3ViIjoid2llbmVyIn0.kQ8-DWBnsHlmIvEvcYDZmBVzpuJ89CeTa4A6zx71n8M:secret1
```

- Yeni bir simetrik anahtar oluştur ve bunu önceki payload'a uygula
- Yeni JWT ile `POST /admin/delete?username=carlos` isteğini yolla

#### 4.Lab: JWT authentication bypass via jwk header injection

Bu labda Jku header'da bir açık olduğu söylenmekte

Adımlar:
- Burp JWT editor ile bir RSA anahtarı oluşturuyoruz
- JWT ile gönderilmiş bir isteği çekiyoruz ve username kısmını administrator olarak değiştiriyoruz
- Ardından önce hazırladığımız embedded key yi kullanarak `POST /admin/delete?username=carlos` isteğini yolluyoruz

#### 5.Lab: JWT authentication bypass via jku header injection

Adımlar:
- Yeni bir RSA anahtarı oluştur
- RSA anahtarını kopyala ve exploit server'a body olarak yapıştır.
- RSA anahtarında "kid" claim'ini `GET /admin` isteğindeki JWT header'ına yaz
- header kısmına bir jku parametresi ekle ve bunu http://your_exploit_server_address/exploit olarak ekler
- RSA anahtarı ile isteğini imzala 
- `GET /admin/delete?username=carlos` isteğini yolla

#### 6.Lab: JWT authentication bypass via kid header path traversal

Adımlar:
- Simetrik bir anahtar oluştur ve k claimini `AA==` olarak değiştir.
- `GET /admin` isteğinin JWTsinin kid değerini `../../../../../../../dev/null` ile değiştir.
- İsteği simetrik anahtar ile imzala
- `GET /admin/delete?username=carlos` isteğini yolla

#### 7.Lab: JWT authentication bypass via algorithm confusion

Adımlar: 
- JWK anahtarını /jwks.json endpoint'ından buluyoruz
- Bunu PEM formatına çevirip base64 ile encode ediyoruz
- Yeni simetrik bir anahtar üretip bunun `k` claim'ini base64 encodeladığımız PEM'i yapıştırıyoruz
- `GET /admin` isteğindeki JWT'nin `alg` claimini `HS256` olarak değiştiriyoruz
- JWT'i simetrik anahtarımızla imzalıyoruz
- `GET /admin?username=carlos` isteğini yeni JWT ile atıyroruz

