Discovery
curl https://static-labs.tryhackme.cloud/sites/favicon/images/favicon.ico | md5sum
robot.txt
sitemap.xml
curl http://MACHINE_IP -v (look into headers)
Framework remnants
wappalyzer
https://archive.org/web/ (wayback hacking)

The format of the S3 buckets is http(s)://{name}.s3.amazonaws.com where {name} is decided by the owner, such as tryhackme-assets.s3.amazonaws.com. S3 buckets can be discovered in many ways, such as finding the URLs in the website's page source, GitHub repositories, or even automating the process. One common automation method is by using the company name followed by common terms such as {name}-assets, {name}-www, {name}-public, {name}-private, etc.

ffuf dirb gobuster

OSINT
sublist3r
https://crt.sh
https://ui.ctsearch.entrust.com/ui/ctsearchui
Brute Force
dnsrecon

Google ,3
site:*.tryhackme.com -site:www.tryhackme.com

visual hosts
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/namelist.txt -H "Host: SomeWebsite" -u http://SomeIP -fs {size}

Subfinder
it is a tool to find the subdomains of a website
subfinder -d hello.com

amass
it is also a tool that finds subdomains
amass enum -d target.com

nslookup
a command line tool that helps you map IP adresses and domain names
nslookup magomedankalaev.com

we can also specify which DNS server to use
nslookup bandage.com 8.8.8.8

you can also check mail exchanges MX
nslookup -query=ns google.com


nslookup
> set type=any 
> example.com