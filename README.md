# Белешки за задачите за интервју

## Задача 1

### 1.1. Пронајдете колку DNS записи за сервери има во него ?

beyondmachines.net има 4 авторитативни DNS сервери

    $nslookup -type=NS beyondmachines.net
    beyondmachines.net      nameserver = ns-2019.awsdns-60.co.uk.
    beyondmachines.net      nameserver = ns-1025.awsdns-00.org.
    beyondmachines.net      nameserver = ns-496.awsdns-62.com.
    beyondmachines.net      nameserver = ns-543.awsdns-03.net.

AWS Route 53 name servers. beyondmachines.net го користи Amazon's DNS service.
Било кој од овие 4 сервери може да одговори на DNS барања за beyondmachines.net

IP адресите за даден name server :

    ns-543.awsdns-03.net    internet address = 205.251.194.31 
    ns-496.awsdns-62.com    internet address = 205.251.193.240
    ns-2019.awsdns-60.co.uk internet address = 205.251.199.227
    ns-1025.awsdns-00.org   internet address = 205.251.196.1


### 1.2 Колку од нив се активни, колку ви изгледаат како заборавени или ранливи.
 * како ги баравте ? 
   * Ги испробав двете комади sublist3r и subfinder за да ги пронајдам subdomain-ите 

          $sublist3r -d beyondmachines.net
          [-] Total Unique Subdomains Found: 13
          www.beyondmachines.net = ACTIVE
          ako.beyondmachines.net = nema DNS zapis
          challenge.beyondmachines.net = ACTIVE
          clarity.beyondmachines.net = nema DNS zapis
          damascian.beyondmachines.net = nema DNS zapis
          damascian-online.beyondmachines.net = nema DNS zapis
          old.beyondmachines.net = postoi DNS zapis, no ne e aktiven
          rendertest.beyondmachines.net = nema DNS zapis
          status.beyondmachines.net = nema DNS zapis
          testalb.beyondmachines.net =  nema DNS zapis
          trust.beyondmachines.net = ACTIVE
          yieldcat.beyondmachines.net = ACTIVE
          yieldhog.beyondmachines.net = nema DNS zapis

Алатката sublist3r открива поддомаини. Собира информации од јавни интернет архиви, web-scraping ...
sublist3r не гарантира дека поддомеините кои ќе ги пронајде се уште активни или конфигурирани, за тие домаини да не чуваме повеќе DNS записи.

### Kаде се хостираат ?

Ја искористив алатката dig, за да ги добијам DNS запис од DNS серверите. За само да ја прикаже IP адресата (A записот) -> +short

Потоа ја искористив алатката curl ipinfo.io/<ip> за да ја видам георрафската локацијата на IP адресита.
Испраќам HTTP барање до  ipinfo.io -> GeoIP база -> назад враќа JSON одговор.

    $ dig beyondmachines.net +short
    18.165.72.128
    18.165.72.2
    18.165.72.91
    18.165.72.16
    
    $ curl ipinfo.io/18.165.72.128     
    {
    "ip": "18.165.72.128",
    "hostname": "server-18-165-72-128.sof50.r.cloudfront.net",
    "city": "Sofia",
    "region": "Sofia-Capital",
    "country": "BG",
    "loc": "42.6975,23.3241",
    "org": "AS16509 Amazon.com, Inc.",
    "postal": "1000",
    "timezone": "Europe/Sofia",
    "readme": "https://ipinfo.io/missingauth"
    }

4 IP addresses, DNS серверот прави load balancing помеѓи овие 4 инстанци кои ја сервираат истата содрзина. Cloud-native features (scaling,rollouts,rollbacks,self-healing, availability zones)


    $ dig ako.beyondmachines.net +short = пример за неактивен domain

---
    $ dig challenge.beyondmachines.net +short
    185.199.111.153
    185.199.110.153
    185.199.108.153
    185.199.109.153
    
    curl ipinfo.io/185.199.111.153
    {
    "ip": "185.199.111.153",
    "hostname": "cdn-185-199-111-153.github.com",
    "city": "San Francisco",
    "region": "California",
    "country": "US",
    "loc": "37.7621,-122.3971",
    "org": "AS54113 Fastly, Inc.",
    "postal": "94107",
    "timezone": "America/Los_Angeles",
    "readme": "https://ipinfo.io/missingauth",
    "anycast": true
    }

---
    $ dig trust.beyondmachines.net +short
    beyondmachines.portals.safebase.io.
    104.18.5.130
    104.18.4.130
    
    $ curl ipinfo.io/104.18.5.130        
    {
    "ip": "104.18.5.130",
    "city": "San Francisco",
    "region": "California",
    "country": "US",
    "loc": "37.7621,-122.3971",
    "org": "AS13335 Cloudflare, Inc.",
    "postal": "94107",
    "timezone": "America/Los_Angeles",
    "readme": "https://ipinfo.io/missingauth",
    "anycast": true
    }      
---
    $ dig yieldcat.beyondmachines.net +short
    216.24.57.1
    
    curl ipinfo.io/216.24.57.1
    {
    "ip": "216.24.57.1",
    "city": "San Francisco",
    "region": "California",
    "country": "US",
    "loc": "37.7621,-122.3971",
    "org": "AS397273 Render",
    "postal": "94107",
    "timezone": "America/Los_Angeles",
    "readme": "https://ipinfo.io/missingauth",
    "anycast": true
    }                            

---
    $ dig old.beyondmachines.net +short  
    192.0.78.233
    192.0.78.189

    $ curl ipinfo.io/192.0.78.233
    
    {
    "ip": "192.0.78.233",
    "city": "San Francisco",
    "region": "California",
    "country": "US",
    "loc": "37.7749,-122.4194",
    "org": "AS2635 Automattic, Inc",
    "postal": "94102",
    "timezone": "America/Los_Angeles",
    "readme": "https://ipinfo.io/missingauth",
    "anycast": true
    }

Што не би требало да е online ?

Испраќање на HTTPS барања:

    $ curl  -I  https://beyondmachines.net
    HTTP/2 405
    content-type: text/html; charset=utf-8
    content-length: 0
    date: Thu, 29 May 2025 10:22:06 GMT
    x-amzn-trace-id: Root=1-6838354e-78cddbfd01799d4a54be2491;Parent=6f9d0da53d17c01e;Sampled=0;Lineage=1:3276204f:0
    x-amzn-requestid: aaa0c967-8f9a-442f-8b93-e9dbabed65fd
    referrer-policy: same-origin
    strict-transport-security: max-age=31536000; includeSubDomains; preload
    allow: GET
    x-amzn-remapped-content-length: 0
    x-frame-options: DENY
    cross-origin-opener-policy: same-origin
    x-amz-apigw-id: LU1EQGHVFiAEcbg=
    x-content-type-options: nosniff
    x-cache: Error from cloudfront
    via: 1.1 69bd99223bbe7be5d36f0fa13d71bf84.cloudfront.net (CloudFront)
    x-amz-cf-pop: SOF50-P1
    x-amz-cf-id: PvGk-42gb2GTgD84oRPsrLK2TtCOCWpQUZyvZBuBr6tLx9Q_WKCYuA==
---

    $ curl -I  https://challenge.beyondmachines.net
    HTTP/2 200
    server: GitHub.com -> sodrzinata e servirana preku GitHub Pages
    content-type: text/html; charset=utf-8
    last-modified: Thu, 27 Feb 2025 12:00:52 GMT
    access-control-allow-origin: *
    etag: "67c053f4-426a"
    expires: Thu, 29 May 2025 10:25:47 GMT
    cache-control: max-age=600
    x-proxy-cache: MISS
    x-github-request-id: 598F:C262A:1AC50B9:1AFBED6:683833D3
    accept-ranges: bytes
    date: Thu, 29 May 2025 10:16:38 GMT
    via: 1.1 varnish
    age: 51
    x-served-by: cache-sof1510028-SOF
    x-cache: HIT
    x-cache-hits: 1
    x-timer: S1748513799.820004,VS0,VE4
    vary: Accept-Encoding
    x-fastly-request-id: aecbde1282b00b4b97d8752a9fc3d8bb0417eb35
    content-length: 17002



### Што по вас не би требало да е online
* Не би требало да се онлајн“:
Поддомени што: 
  * Немаат IP.
    * Не се моментално активни поддомаини
      * old.beyondmachines.net =>  SSL сертификат mismatch.
        Серверот на IP адресата нема SSL сертификат што важи за old.beyondmachines.net.

            $ curl -I https://old.beyondmachines.net      
            curl: (60) SSL: certificate subject name (wordpress.com) does not match target host name 'old.beyondmachines.net'
            More details here: https://curl.se/docs/sslcerts.html
    
            curl failed to verify the legitimacy of the server and therefore could not
            establish a secure connection to it. To learn more about this situation and
            how to fix it, please visit the web page mentioned above.

      * Претходно постоел и бил регистриран на WordPress инфраструктурата.
      * Сега не е активен, но сеуште имаме DNS запис за него. 
      * серверот враќа општ SSL сертификат (за wordpress.com). Треба да е конфигуриран со свој SSL сертификат.

* Активни:
 * Поддомени со IP + успешен HTTP(S) одговор.
 * www.beyondmachines.net
 * challenge.beyondmachines.net
 * trust.beyondmachines.net
 * yieldcat.beyondmachines.net

Проверка со командата : curl -I https://beyondmachines.net , ова е пример за HTTP барање so method: HEAD,  враќа назад HTTP headers, само да се провери дали страницата постои.
Но ова не беше успешно бидејќи серверот само дозволува HTTP GET барања.

Затоа може да се тестира во browser или без -I flag-ot.

-----
## Задача 2

* 45.33.49.201 : **Има успешен обид за пробивање на серверот. Се логира како улога на admin.** 
  * бара да го пристапи robots.txt, кои страници/патеки не треба да ги индексираат ботовите, но серверот го нема овој фајл, ботот (се претставува како Googlebot) ќе ги скенираат сите страници без ограничувања.
  * бара да скенира сензитивни фајлови .git, .env, но серверот му враќа 404 (Not Found) -> не успева
  * бара да скенира phymyadmin i backup files, но серверот му враќа 404 (Not Found) -> не успева
  * сака да пристапи на login form за админ панел, GET admin/login.php -> серверот му враќа 200 (OK) -> успева да ја пристапи страната
  * се обидува неколку пати да се најави на формата, но не успева -> серверот враќа 401 (Unauthorized)
  * последниот обид да се најави на формата е успешен, серверот враќа статус 302 (Found) -> го redirect-ира на друга URL address
  * страната на која го редиректира е /admin/dashboard.php, успешно влегов во админ панелот.
  * вристапува до /admin/users.php, /admin/settings.php, /admin/backup.php, 
  * пристапува до /admin/uploads.php, испраќа POST барање, тоа е успешно.
  * го гледа филе-от /uploads.shell.php, ова е многу опасно, може целосна контрола над серверот да има, да извршува команди во серверот
  * Се одлогира.
  * Потоа повторно се најавува успешно како админ корисник.
  * Прави промена на информациите за корисник кој е зачуван со id=1 во базата

Начини за да се превенира ова:
* Ограничување на обиди (brute force protection) = ограничи обиди за неуспешни најави
* Користење на HTTPS, лозинките се праќаат не енкриптирана и може да биде направен sniffing.
* 2FA (дво-факторска автентикација), дури и да влезе некој ез вторниот фактор нема да може да пристапи.
* Да направиме конфигурирање правила на firewall-от, за забрана на одредени IP адреси ако приметиме дека пробуваат активно да пристапат