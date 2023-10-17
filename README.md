# **Jarkom Practicum Module 2 Group I06*

### **Members of the group**

| **Name**                  | **NRP**    |
| ------------------------- | ---------- |
| Ahmad Danindra Nugroho    | 5025211259 |
| Kayla Angelica Priambudi  | 5025211262 |

## Daftar Isi

1. [Soal](#Soal)
2. [Setup-Topologi](#Setup-Topologi)
3. [Setup-DNS](#Setup-DNS)
4. [Setup-WebServer](#Setup-WebServer)
5. [Setting-WebServer](#Setting-WebServer)
6. [Bash-Scripts](#Bash-Scripts)

## Soal

1. Yudhistira akan digunakan sebagai DNS Master, Werkudara sebagai DNS Slave, Arjuna merupakan Load Balancer yang terdiri dari beberapa Web Server yaitu Prabakusuma, Abimanyu, dan Wisanggeni. Buatlah topologi dengan pembagian sebagai berikut. Folder topologi dapat diakses pada drive berikut
2. Buatlah website utama pada node arjuna dengan akses ke **arjuna.yyy.com** dengan alias **www.arjuna.yyy.com** dengan yyy merupakan kode kelompok.
3. Dengan cara yang sama seperti soal nomor 2, buatlah website utama dengan akses ke **abimanyu.yyy.com** dan alias **www.abimanyu.yyy.com**.
4. Kemudian, karena terdapat beberapa web yang harus di-deploy, buatlah subdomain **parikesit.abimanyu.yyy.com** yang diatur DNS-nya di Yudhistira dan mengarah ke Abimanyu.
5. Buat juga reverse domain untuk domain utama. (_Abimanyu saja yang direverse_)
6. Agar dapat tetap dihubungi ketika DNS Server Yudhistira bermasalah, buat juga Werkudara sebagai DNS Slave untuk domain utama.
7. Seperti yang kita tahu karena banyak sekali informasi yang harus diterima, buatlah subdomain khusus untuk perang yaitu **baratayuda.abimanyu.yyy.com** dengan alias **www.baratayuda.abimanyu.yyy.com** yang didelegasikan dari Yudhistira ke Werkudara dengan IP menuju ke Abimanyu dalam folder Baratayuda.
8. Untuk informasi yang lebih spesifik mengenai Ranjapan Baratayuda, buatlah subdomain melalui Werkudara dengan akses **rjp.baratayuda.abimanyu.yyy.com** dengan alias **www.rjp.baratayuda.abimanyu.yyy.com** yang mengarah ke Abimanyu.
9. Arjuna merupakan suatu Load Balancer Nginx dengan tiga worker (yang juga menggunakan nginx sebagai webserver) yaitu Prabakusuma, Abimanyu, dan Wisanggeni. Lakukan deployment pada masing-masing worker.
10. Kemudian gunakan algoritma **Round Robin** untuk Load Balancer pada **Arjuna**. Gunakan _server_name_ pada soal nomor 1. Untuk melakukan pengecekan akses alamat web tersebut kemudian pastikan worker yang digunakan untuk menangani permintaan akan berganti ganti secara acak. Untuk webserver di masing-masing worker wajib berjalan di port 8001-8003. Contoh
    - _Prabakusuma:8001_
    - _Abimanyu:8002_
    - _Wisanggeni:8003_
11. Selain menggunakan Nginx, lakukan konfigurasi Apache Web Server pada worker Abimanyu dengan web server **www.abimanyu.yyy.com**. Pertama dibutuhkan web server dengan DocumentRoot pada /var/www/abimanyu.yyy
12. Setelah itu ubahlah agar url **www.abimanyu.yyy.com/index.php/home** menjadi **www.abimanyu.yyy.com/home**.
13. Selain itu, pada subdomain **www.parikesit.abimanyu.yyy.com**, DocumentRoot disimpan pada /var/www/parikesit.abimanyu.yyy
14. Pada subdomain tersebut folder /public hanya dapat melakukan _directory listing_ sedangkan pada folder /secret tidak dapat diakses (_403 Forbidden_).
15. Buatlah kustomisasi halaman error pada folder /error untuk mengganti error kode pada Apache. Error kode yang perlu diganti adalah 404 Not Found dan 403 Forbidden.
16. Buatlah suatu konfigurasi virtual host agar file asset **www.parikesit.abimanyu.yyy.com/public/js** menjadi **www.parikesit.abimanyu.yyy.com/js**
17. Agar aman, buatlah konfigurasi agar **www.rjp.baratayuda.abimanyu.yyy.com** hanya dapat diakses melalui port 14000 dan 14400.
18. Untuk mengaksesnya buatlah autentikasi username berupa “Wayang” dan password “baratayudayyy” dengan yyy merupakan kode kelompok. Letakkan DocumentRoot pada /var/www/rjp.baratayuda.abimanyu.yyy.
19. Buatlah agar setiap kali mengakses IP dari Abimanyu akan secara otomatis dialihkan ke **www.abimanyu.yyy.com (alias)**
20. Karena website **www.parikesit.abimanyu.yyy.com** semakin banyak pengunjung dan banyak gambar gambar random, maka ubahlah request gambar yang memiliki substring “abimanyu” akan diarahkan menuju abimanyu.png.

PS:

- **yyy** pada url adalah **kode kelompok anda**
- File requirement dapat diakses melalui drive berikut.

## Configuration of Network Topology

The network topology has been established in accordance with the provided instructions. In this setup, NAT1 is linked to the Router, and the Router is further connected to two switches. Each switch is then connected to various nodes. Specifically, `Switch1` is associated with `NakulaClient`,` SadewaClient`, `AbimanyuWebServer`, `PrabakusumaWebServer`, and `WisanggeniWebServer`. On the other hand, `Switch2 `is connected to `YudhistiraDNSMaster`, `WerkudaraDNSSlave`, and `ArjunaLoadBalancer`.

> _Answer Question Number 1_

![topologi](./assets/topologi.png)

The following is a list of IPs for each node in the topology above

```sh
NakulaClient = 192.231.1.2
SadewaClient = 192.231.1.3
AbimanyuWebServer = 192.231.1.4
PrabukusumaWebServer = 192.231.1.5
WisanggeniWebServer = 192.231.1.6

YudhistiraDNSMaster = 192.231.2.2
WerkudaraDNSSlave = 192.231.2.3
ArjunaLoadBalancer = 192.231.2.4
```

## Setup-DNS

DNS will be setup in YudhistiraDNSMaster and WerkudaraDNSSlave. The requirements for the question are as follows:

YudhistiraDNSMaster and WerkudaraDNSSlave will be configured with DNS. The following are the question's prerequisites:

- Create a main website on the Arjuna node with access to arjuna.yyy.com and the alias www.arjuna.yyy.com where yyy is the group code (question 2)
- Create a main website with access to abimanyu.yyy.com and the alias www.abimanyu.yyy.com (question 3)
- Create a subdomain parikesit.abimanyu.yyy.com whose DNS is set at Yudhistira and pointing to Abimanyu (question 4)
- Create a reverse domain for the main domain as well (question 5).
- Add Werkudara as a DNS Slave for the primary domain (question 6).
- Create a special subdomain for war, namely baratayuda.abimanyu.yyy.com with the alias www.baratayuda.abimanyu.yyy.com which is delegated from Yudhistira to Werkudara with the IP going to Abimanyu in the Baratayuda folder <span style="color:orange; font- weight:bold;">(question 7)</span>
- Create a subdomain via Werkudara with access rjp.baratayuda.abimanyu.yyy.com with the alias www.rjp.baratayuda.abimanyu.yyy.com which points to Abimanyu <span style="color:orange; font-weight:bold;"> (question 8)</span>

With these provisions, first we have to set DNS in Yudhistira as DNS Master.

### Setting DNS Master

> _named.conf.local_ Yudhistira

![namedconflocal-master](./assets/namedconflocal-master.png)

```bind
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

// main arjuna
zone "arjuna.d15.com" {
        type master;
        notify yes;
        also-notify { 192.231.2.3; };  // IP Werkudara sebagai slave
        allow-transfer { 192.231.2.3; };  // IP Werkudara sebagai slave
        file "/etc/bind/arjuna/arjuna.i06.com";
};

// main abimanyu
zone "abimanyu.d15.com" {
        type master;
        notify yes;
        also-notify { 192.231.2.3; };  // IP Werkudara sebagai slave
        allow-transfer { 192.231.2.3; };  // IP Werkudara sebagai slave
        file "/etc/bind/abimanyu/abimanyu.i06.com";
};

zone "baratayuda.abimanyu.i06.com" {
    type master;
    file "/etc/bind/delegasi/baratayuda.abimanyu.i06.com";
    allow-transfer { 192.231.2.3; };  // IP address of the slave server
};

// reverse abimanyu
zone "1.231.192.in-addr.arpa" {
    type master;
    notify yes;
    also-notify { 192.231.2.3; };  // IP Werkudara sebagai slave
    allow-transfer { 192.231.2.3; };  // IP Werkudara sebagai slave
    file "/etc/bind/abimanyu/1.231.192.in-addr.arpa";
};
```

In the BIND DNS server configuration file `named.conf.local`, we have setup several DNS zones, including:

1. `arjuna.i06.com` : DNS zone for the **arjuna.i06.com** domain, set as the master server, and notifies the slave server (Werkudara) when there are updates.
2. `abimanyu.i06.com` : DNS zone for the domain **abimanyu.i06.com**, also set as the master server, and notifies the slave server (Werkudara) when changes are made.
3. `baratayuda.abimanyu.i06.com` : DNS zone for the subdomain **baratayuda.abimanyu.i06.com**, set as the master server, and zone data for the subdomain is stored in a file.
    - <span style="color:gray; font-style:italic;"> note: for this zone, this is a tight solution because if there is no zone declaration, the Baratayuda subdomain will not be read as a subdomain delegation to Werkudara. (not really best practice)</span>
4. `1.231.192.in-addr.arpa` : reverse DNS zone for the IP address range 192.231.1.0/24 used by the domain abimanyu.i06.com, also set as the master server, and notifies the slave server (Werkudara) when there is a change.

After that, the zone BIND files for each zone are as follows.
> _Arjuna Zone_

![arjunazone](./assets/arjunazone.png)

```bind
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     arjuna.i06.com. root.arjuna.i06.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      arjuna.i06.com.
@       IN      A       192.231.2.4       ; IP Arjuna
www     IN      CNAME   arjuna.i06.com.
@       IN      AAAA    ::1
```

In the first line, **NS (Name Server)** indicates that the authoritative server for this zone is `arjuna.i06.com`. Then, the **A (Address)** record associates the name **@ (root domain)** with the IP address `192.231.2.4`, possibly referring to this DNS server. Additionally, there is a **CNAME** record that associates `www with arjuna.i06.com`, so that when accessing `www.arjuna.i06.com`, it refers to `arjuna.i06.com`.

> _Abimanyu Zone_

![abimanyuzone](./assets/abimanyuzone.png)

```bind
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     abimanyu.i06.com. root.abimanyu.i06.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      abimanyu.i06.com.
@       IN      A       192.231.1.4       ; IP Abimanyu
www     IN      CNAME   abimanyu.d15.com.
parikesit       IN      A       192.231.1.4       ; IP Abimanyu
www.parikesit   IN      CNAME   parikesit.abimanyu.i06.com.
ns1     IN      A       192.231.2.3       ; IP Werkudara
baratayuda      IN      NS      ns1
@       IN      AAAA    ::1
```

The **NS (Name Server)** declaration indicates that the authoritative server for this zone is `abimanyu.di06.com`. The **A (Address)** record associates the name **@ (root domain)** with the IP address **192.231.1.4**, which may refer to this DNS server. Additionally, there is a **CNAME** record that associates www with `abimanyu.i06.com`, so that when accessing `www.abimanyu.i06.com`, it refers to `abimanyu.i06.com`. There is also another **A** record for **subdomain parikesit** that points to the same IP address as `abimanyu.i06.com`. Then, the **CNAME** record for `www.parikesit` associates that subdomain with `parikesit.abimanyu.i06.com`. Next, there is a delegation of the subdomain `baratayuda.abimanyu.i06.com` with an NS record that directs to the server `ns1 (Werkudara)`. This allows the `baratayuda` subdomain to be managed by a different DNS server, in this case, the ns1 server. There is also an AAAA record associating the name **@** with the **IPv6 loopback ::1** address.

> _Reverse Abimanyu Zone_

![reverseabimanyuzone](./assets/reverseabimanyuzone.png)

```bind
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     abimanyu.i06.com. root.abimanyu.i06.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
192.231.10.in-addr.arpa.IN      NS      abimanyu.i06.com.
4                       IN      PTR     abimanyu.i06.com. ; Byte ke-4 Abimanyu
```

The NS (Name Server) declaration connects this reverse zone to the server `abimanyu.i06.com`. Next, there is a **PTR** (Pointer) record that connects the IP address `192.231.1.4` with the domain name `abimanyu.i06.com`.

So that the delegation of the subdomain `baratayuda.abimanyu.i06.com` can work, we have to change a few settings in `named.conf.options`

![namedconfoptions-master](./assets/namedconfoptions-master.png)

```bind
options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        forwarders {
               192.168.122.1; // IP ns Router
        };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        //dnssec-validation auto;
        allow-query{any;};

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
```

Adding `allow-query{any;};` allows the DNS server to accept various types of DNS requests, including requests related to subdomain delegation, allowing the flow of delegation information. Apart from that, IP forwarding is also implemented to the router so that clients can access the internet simply using the Master nameserver.

### Setting DNS Slave

> _named.conf.local_ Werkudara

![namedconflocal-slave](./assets/namedconflocal-slave.png)

```bind
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "arjuna.i06.com" {
    type slave;
    masters { 192.231.2.2; }; // IP Yudhistira sebagai Master
    file "var/lib/bind/arjuna/arjuna.i06.com";
};

zone "abimanyu.i06.com" {
    type slave;
    masters { 192.231.2.2; }; // IP Yudhistira sebagai Master
    file "var/lib/bind/abimanyu/abimanyu.i06.com";
};

zone "1.231.192.in-addr.arpa" {
    type slave;
    masters { 192.231.2.2; };  // IP Yudhistira sebagai Master
    file "/etc/lib/bind/abimanyu/1.231.192.in-addr.arpa";
};

zone "baratayuda.abimanyu.i06.com" {
    type master;
    file "/etc/bind/delegasi/baratayuda.abimanyu.i06.com";
};
```

In this configuration, there are several zones in the DNS server.

- Zones `"arjuna.i06.com"` and `"abimanyu.i06.com"` are defined as **slave zones** which follow the master server (Yudhistira) with the aim of storing a copy of the master zone data.
- Zone `"1.231.192.in-addr.arpa"` is also a **slave zone** that follows the master server for **reverse DNS resolution**.
- Zone `"baratayuda.abimanyu.i06.com"` is specified as zone master, which means this DNS server is authoritative for this zone and is responsible for delegation of subdomain `"baratayuda.abimanyu.i06.com"`.

After that, the zone BIND file for the Baratayuda delegation zone is as follows.

> _Barayauda Zone Delegation_

![baratayudazone](./assets/baratayudazone.png)

```bind
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     baratayuda.abimanyu.i06.com. root.baratayuda.abimanyu.i06.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      baratayuda.abimanyu.i06.com.
@       IN      A       192.231.1.4       ; IP Abimanyu (baratayuda)
www     IN      CNAME   @               ; IP Abimanyu (baratayuda)
rjp     IN      A       192.231.1.4       ; IP Abimanyu (rjp baratayuda)
www.rjp IN      CNAME   rjp.baratayuda.abimanyu.i06.com. ; IP Abimanyu (rjp baratayuda)
```

The **NS (Name Server)** declaration connects this zone to an authoritative server, namely `baratayuda.abimanyu.i06.com.` which points to the IP address `192.231.1.4`. Furthermore, record **A** associates the IP address `192.231.1.4` with the domain name `baratayuda.abimanyu.i06.com`. Then, there is a **CNAME** record which creates the alias `www` to refer to the zone `baratayuda.abimanyu.i06.com` and `rjp` which points to the IP address `192.231.1.4`. Finally, another **CNAME** record associates the alias `www.rjp` with the subdomain `rjp.baratayuda.abimanyu.i06.com` in the zone `baratayuda.abimanyu.i06.com`.

So that the delegation of the subdomain `baratayuda.abimanyu.i06.com` can work, we have to change a few settings in `named.conf.options`
![namedconfoptions-slave](./assets/namedconfoptions-slave.png)

```bind
options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        // forwarders {
        //      0.0.0.0;
        // };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        //dnssec-validation auto;
        allow-query{any;};

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
```

The settings in this file are the same as the master, only IP forwarding is not applied to the router for the internet here.

So that all of this runs automatically when the node is turned on and progress is not lost, do the following steps

- Copy the bind folder to root with the command `cp -rf /etc/bind /root/dns`
- Create a script to install the dependencies and commands needed for bind9
  
```sh
apt-get update
apt-get install bind9 -y
cp -r -f /root/dns/bind /etc/
service bind9 restart
```

- Call the script via **/root/.bashrc**

The following are the testing results from NakulaClient

![nakula-ping](./assets/nakula-ping.png)

## Setup-WebServer

The web server will be setup on ArjunaLoadBalancer, AbimanyuWebServer, PrabakusumaWebServer, and WisanggeniWebServer. The requirements for the question are as follows:

- Arjuna is an Nginx Load Balancer with three workers (which also use Nginx as a webserver) namely Prabakusuma, Abimanyu, and Wisanggeni. <span style="color:orange; font-weight:bold;">(question 9)</span>
- Use the Round Robin algorithm for Load Balancer on Arjuna. Use server_name in question number 1. The webserver in each worker must run on port 8001-8003. Example: <span style="color:orange; font-weight:bold;">(question 10)</span>
   - _Prabakusuma:8001_
   - _Abimanyu:8002_
   - _Wisanggeni:8003_

First of all, we have to setup the nginx Load Balancer on the Arjuna node, which applies the **Round Robbin** algorithm in its load balancing.

### Load Balancer Setting

![nginx-lbarjuna](./assets/nginx-lbarjuna.png)

```nginx
# Round Robin
upstream myweb  {
        server 192.231.1.5:8001; #IP Abimanyu
        server 192.231.4:8002; #IP Prabakusuma
        server 192.231.1.6:8003; #IP Wisanggeni
}

 server {
        listen 80;
        server_name arjuna.i06.com;

        location / {
        proxy_pass http://myweb;
        }
}
```

- `upstream` - defines the group or group of servers that will receive traffic or load with myweb as the upstream name. In this case, there are two servers with IP addresses respectively server 192.231.1.5:8001, server 192.231.1.4:8002 and server 192.231.1.6:8003. With the Round Robin method, traffic will be divided equally between these two servers.
- `listen` - defines that this web server will run and listen for requests on port 80.
- `location / { ... }` sets how the Nginx server handles requests. In this case, all requests will be forwarded to the server group defined in the upstream section.

After implementing these settings, don't forget to create a symlink with `ln -s /etc/nginx/sites-available/lb-arjuna /etc/nginx/sites-enabled` and restart nginx with `service nginx restart`.

### Worker Settings

After the load balancer has been set, we will set the nginx for each worker.

#### Abimanyu

On the Abimanyu server, a file will be created in the `sites-available` directory named `arjuna` with the following configuration.

![nginx-arjuna-abimanyu](./assets/nginx-arjuna-abimanyu.png)

```nginx
server {

        listen 8002;

        root /var/www/arjuna.i06.com;

        index index.php;
        server_name _;

        location / {
                        try_files $uri $uri/ /index.php?$query_string;
        }

        # pass PHP scripts to FastCGI server
        location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
        }

location ~ /\.ht {
                        deny all;
        }

        error_log /var/log/nginx/jarkom_error.log;
        access_log /var/log/nginx/jarkom_access.log;
}
```

In this configuration the nginx server will run on port `8002`.

- Points to the root directory at `/var/www/arjuna.i06.com`.
- Use `index.php` as index file.
- Uses `server_name _;` placeholder to handle all incoming requests.
- Uses the location / block to configure basic routes, trying to find a matching file or redirecting to index.php with a query string if it doesn't exist.
- Uses the location ~ \.php$ block to send PHP requests to the FastCGI server.
- Ignores .htaccess files with the location ~ /\.ht block and prohibits access to them.
- Record error logs to /var/log/nginx/jarkom_error.log.
- Record access logs to /var/log/nginx/jarkom_access.log.

After implementing this, a symlink must be created with `ln -s /etc/nginx/sites-available/arjuna /etc/nginx/sites-enabled` then restart nginx with `service nginx restart`. Because abimanyu will also run apache2 for the abimanyu web, the default file must be removed from sites-enabled because the port will collide with the apache2 port (80). `rm -rf /etc/nginx/sites-enabled/default`

After configuring nginx, we have to download the index.php file from the drive link that was shared in the question. The steps are as follows

- Download the file using the following wget command

```bash
wget --load-cookies /tmp/cookies.txt "https://docs.google.com/uc?export=download&confirm=$(wget --quiet --save-cookies /tmp/cookies.txt --keep-session-cookies --no-check-certificate 'https://docs.google.com/uc?export=download&id=17tAM_XDKYWDvF-JJix1x7txvTBEax7vX' -O- | sed -rn 's/.*confirm=([0-9A-Za-z_]+).*/\1\n/p')&id=17tAM_XDKYWDvF-JJix1x7txvTBEax7vX" -O arjuna.zip && rm -rf /tmp/cookies.txt
```

- Unzip the file with the output folder name `arjuna.i06.com`.

#### Prabakusuma and Wisanggeni

The nginx configuration on Prabakusuma and Wisanggeni will be the same as that on Abimanyu. There's just no need to delete the default files from sites-enabled because it doesn't run apache2.

> Prabakusuma nginx configuration

![nginx-arjuna-prabakusuma](./assets/nginx-arjuna-prabakusuma.png)

```nginx
server {

        listen 8001;

        root /var/www/arjuna.d15.com;

        index index.php;
        server_name _;

        location / {
                        try_files $uri $uri/ /index.php?$query_string;
        }

        # pass PHP scripts to FastCGI server
        location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
        }

location ~ /\.ht {
                        deny all;
        }

        error_log /var/log/nginx/jarkom_error.log;
        access_log /var/log/nginx/jarkom_access.log;
}
```

>  Nginx Wisanggeni Configuration

![nginx-arjuna-wisanggeni](./assets/nginx-arjuna-wisanggeni.png)

```nginx
server {

        listen 8003;

        root /var/www/arjuna.i06.com;

        index index.php;
        server_name _;

        location / {
                        try_files $uri $uri/ /index.php?$query_string;
        }

        # pass PHP scripts to FastCGI server
        location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
        }

location ~ /\.ht {
                        deny all;
        }

        error_log /var/log/nginx/jarkom_error.log;
        access_log /var/log/nginx/jarkom_access.log;
}
```

The following are the results of testing with `lynx.arjuna.i06.com`

![nakula-lynx-arjuna](./assets/nakula-lynx-arjuna.png)

If you spam Lynx, the destination server will change. If you want to test success, you can turn off nginx on all worker servers other than the destination server you want to test.

## Settings-WebServer

The webserver will be setup on AbimanyuWebServer. The requirements for the question are as follows:

- Apart from using Nginx, configure the Apache Web Server on the Abimanyu worker with the web server www.abimanyu.yyy.com. First you need a web server with DocumentRoot at /var/www/abimanyu.yyy <span style="color:orange; font-weight:bold;">(question 11)</span>
- After that, change the url www.abimanyu.yyy.com/index.php/home to www.abimanyu.yyy.com/home. <span style="color:orange; font-weight:bold;">(question 12)</span>
- Additionally, on the subdomain www.parikesit.abimanyu.yyy.com, DocumentRoot is stored at /var/www/parikesit.abimanyu.yyy <span style="color:orange; font-weight:bold;">(question 13) </span>
- On this subdomain the /public folder can only do directory listing while the /secret folder cannot be accessed (403 Forbidden). <span style="color:orange; font-weight:bold;">(question 14)</span>
- Create a customized error page in the /error folder to replace the error code in Apache. The error codes that need to be changed are 404 Not Found and 403 Forbidden. <span style="color:orange; font-weight:bold;">(question 15)</span>
- Create a virtual host configuration so that the asset file www.parikesit.abimanyu.yyy.com/public/js becomes www.parikesit.abimanyu.yyy.com/js <span style="color:orange; font-weight:bold;" >(question 16)</span>
- To be safe, configure it so that www.rjp.baratayuda.abimanyu.yyy.com can only be accessed via ports 14000 and 14400. <span style="color:orange; font-weight:bold;">(question 17)</ span>
- To access it, create an authenticated username in the form of "Wayang" and password "baratayudayyy" where yyy is the group code. Place DocumentRoot in /var/www/rjp.baratayuda.abimanyu.yyy. <span style="color:orange; font-weight:bold;">(question 18)</span>
- Make it so that every time you access the IP from Abimanyu you will automatically be redirected to www.abimanyu.yyy.com (aka) <span style="color:orange; font-weight:bold;">(question 19)</span>
- Because the website www.parikesit.abimanyu.yyy.com is getting more and more visitors and lots of random images, change image requests that have the substring "abimanyu" to be directed to abimanyu.png. <span style="color:orange; font-weight:bold;">(question 20)</span>

> _Installing Apache2 and PHP_

install apache2 on the abimanyu web server

```
$ apt-get update && apt-get install apache2 -y
```

Don't forget to also install PHP

```
$ apt-get install php libapache2-mod-php7.0 -y
```

After that, the web server configuration for each FQDN is as follows.

> _abimanyu.i06.com_

create a file at the location <code>/etc/apache2/sites-available/abimanyu.i06.conf</code> then fill in the following:

```
<VirtualHost *:80>

   ServerName abimanyu.i06.com
   ServerAlias www.abimanyu.i06.com
   ServerAdmin webmaster@localhost
   DocumentRoot /var/www/abimanyu.i06

</VirtualHost>
```

After that, we enable the script with the following command

```
$ a2ensite /etc/apache2/sites-available/abimanyu.i06.conf
```

After that, we change the URL www.abimanyu.yyy.com/index.php/home to www.abimanyu.yyy.com/home.

we add the following in <code>/etc/apache2/sites-available/abimanyu.i06.conf</code>

```
<Directory /var/www/abimanyu.i06>
    Options +FollowSymLinks -Multiviews
    AllowOverride All
</Directory>
```

we also add the following code in <code>/var/www/abimanyu.i06/.htaccess</code>

```
RewriteEngine On
RewriteCond ${REQUEST_FILENAME} !-d
RewriteRule ^([^\.]+)$ $1.html [NC,L]
```

The code above will remove <code>\*.html</code> from all requests, so we can make requests to <code>http://abimanyu.i06.com/home</code> because the index page is in the file with the location <code>/var/www/abimanyu.i06/home.html</code>

Don't forget to enable the rewrite module so that the code above can function with the following command

```
$ a2enmod rewrite
```

so that every time you access the IP from Abimanyu it will automatically be redirected to www.abimanyu.yyy.com, we add the following code in 000-default.conf

```
<VirtualHost *:80>

    ServerName 192.231.1.4
    Redirect permanent / http://www.abimanyu.i06.com/
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/abimanyu.d15.com

</VirtualHost>
```

restart apache2

```
$ service apache2 restart
```

> _parikesit.abimanyu.d15.com_

create a file at the location <code>/etc/apache2/sites-available/parikesit.abimanyu.i06.conf</code> then fill in the following:

```
<VirtualHost *:80>

    ServerName parikesit.abimanyu.i06.com
    ServerAka www.parikesit.abimanyu.i06.com
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/parikesit.abimanyu.i06

</VirtualHost>
```

After that, we enable the script with the following command

```
$ a2ensite /etc/apache2/sites-available/parikesit.abimanyu.d15.conf
```

Then, we make it so that on the subdomain the /public folder can only do directory listing while the /secret folder cannot be accessed (403 Forbidden) by adding the following script in <code>/etc/apache2/sites-available/parikesit.abimanyu. i06.conf</code>

```
<Directory /var/www/parikesit.abimanyu.i06/public>
         Options +Indexes
</Directory>

<Directory /var/www/parikesit.abimanyu.i06/secret>
         Options -Indexes
</Directory>
```

To customize the error page in the /error folder to replace the error code in Apache. The error codes that need to be replaced are 404 Not Found and 403 Forbidden, we need to add the following script in <code>/etc/apache2/sites-available/parikesit.abimanyu.i06.conf</code>

```
ErrorDocument 404 /error/404.html
ErrorDocument 403 /error/403.html
```

so that the asset file <code>www.parikesit.abimanyu.yyy.com/public/js</code> becomes
<code>www.parikesit.abimanyu.yyy.com/js</code>, we can add the following code in <code>/etc/apache2/sites-available/parikesit.abimanyu.i06.conf</code>

```
Alias "/js" "/var/www/parikesit.abimanyu.i06.com/public/js"
```

Because the website www.parikesit.abimanyu.yyy.com is getting more and more visitors and lots of random images, change image requests that have the substring "abimanyu" to be directed to abimanyu.png. To do this, we need to add the following code in <code>/etc/apache2/sites-available/parikesit.abimanyu.i06.conf</code>

```
RewriteEngine On
RewriteCond ${REQUEST_URI} abimanyu
RewriteRule \.(jpg|jpeg|png|gif)$ /public/images/abimanyu.png [L]
```

So that the script above can be run, we need to activate the rewrite module with the following command

```
$ a2enmod rewrite
```

restart apache2

```
$ service apache2 restart
```

> _rjp.baratayuda.abimanyu.d15.com_

To create a configuration so that www.rjp.baratayuda.abimanyu.yyy.com can only be accessed via ports 14000 and 14400, we need to create the following script in <code>/etc/apache2/sites-available/rjp.baratayuda.abimanyu.i06 .com</code>

```
<VirtualHost *:14000 *:14400>

    ServerName rjp.baratayuda.abimanyu.i06.com
    ServerAka www.rjp.baratayuda.abimanyu.i06.com
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/rjp.baratayuda.abimanyu.i06

    <Directory /var/www/rjp.baratayuda.abimanyu.i06.com>
         AuthType Basic
         AuthName "Restricted Area"
         AuthUserFile /etc/apache2/.htpasswd
         Require valid-user
    </Directory>

</VirtualHost>
```

After that, we enable the script with the following command

```
$ a2ensite /etc/apache2/sites-available/rjp.baratayuda.abimanyu.i06.conf
```

After that, we need to create a file <code>/etc/apache2/.htpasswd</code> as a place to set the password and username with the following command

```
$ htpasswd -bc /etc/apache2/.htpasswd Wayang baratayudad15
```

then we enable the modules needed to be able to log in using the username and password with the following command
```
$ a2enmod authn_core authz_core authn_file authz_user
```

restart apache2

```
$ service apache2 restart
```

> _Testing_

To check, we go to NakulaClient and install Lynx

```
$ apt-get update && apt-get install lynx -y
```

11. www.abimanyu.i06.com

```
$ lynx www.abimanyu.i06.com
```

Results:
<img src="assets/webserver/soal_11.png" />

12. www.abimanyu.yyy.com/home

```
$ lynx www.abimanyu.i06.com/home
```

Results:
<img src="assets/webserver/soal_11.png" />

13. www.parikesit.abimanyu.yyy.com

```
$ lynx www.parikesit.abimanyu.yyy.com
```

Results:
<img src="assets/webserver/soal_13.png" />

14. The /public folder can only do directory listing while the /secret folder cannot be accessed (403 Forbidden)

```
$ lynx parikesit.abimanyu.i06.com/secret
```

Results:
<img src="assets/webserver/soal_14.png" />

15. Create a customized error page in the /error folder to replace the error code in Apache. The error codes that need to be changed are 404 Not Found and 403 Forbidden.

```
$ lynx parikesit.abimanyu.i06.com/secret
```

Results:
<img src="assets/webserver/soal_14.png" />

```
$ lynx parikesit.abimanyu.i06.com/testing
```

Results:
<img src="assets/webserver/not_found.png" />

16. www.parikesit.abimanyu.yyy.com/public/js be
     www.parikesit.abimanyu.yyy.com/js

```
$ lynx parikesit.abimanyu.i06.com/js
```

Results:
<img src="assets/webserver/js.png" />

17. www.rjp.baratayuda.abimanyu.yyy.com can only be accessed via ports 14000 and 14400.

```
$ lynx rjp.baratayuda.abimanyu.i06.com:14000
```

or

```
$ lynx rjp.baratayuda.abimanyu.i06.com:14400
```

Results:
<img src="assets/webserver/js.png" />

```
$ lynx rjp.baratayuda.abimanyu.i06.com:8900
```

Results:
<img src="assets/webserver/rjp_unavail.png" />

18. create an authenticated username in the form of "Wayang" and password "baratayudayyy" with yyy being the group code

```
$ lynx rjp.baratayuda.abimanyu.i06.com:14000
```

or

```
$ lynx rjp.baratayuda.abimanyu.i06.com:14400
```

Results:
<img src="assets/webserver/username.png" />

<img src="assets/webserver/password.png" />

<img src="assets/webserver/rjp_baratayuda.png" />

19. Make it so that every time you access the IP from Abimanyu it will automatically be redirected to www.abimanyu.yyy.com

```
$ lynx 192.231.1.4
```

Results:
<img src="assets/webserver/soal_11.png" />

20. Because the website www.parikesit.abimanyu.yyy.com is getting more and more visitors and lots of random images, change image requests that have the substring "abimanyu" to be directed to abimanyu.png.

To check whether the downloaded image is <code>abimanyu.png</code> we install exiftool with the following command in NakulaClient

```
$ apt-get install exiftool -y
```

we try to download the image

```
$ lynx parikesit.abimanyu.i06.com/public/images/abimanyu.png
```

If you look at the following are the filen specifications:

```
$exiftool abimanyu.png
```

<img src="assets/webserver/abimanyu_png.png">

we try to download another file containing the substring "abimanyu"

```
$ lynx parikesit.abimanyu.d15.com/public/images/not-abimanyu.png
```

If you look at the following are the filen specifications:

```
$exiftool abimanyu.png
```

<img src="assets/webserver/not_abimanyu_png.png">

here we can see that even though the name is different, the specifications are exactly the same which means what is downloaded is the abimanyu.png file

## Bash Scripts
The following is a snippet of the script that is run by each node to restore and run all the installations and configurations that have been carried out. The key is that we usually backup core folders such as `/etc/bind`, `/etc/nginx`, `/etc/apache2`, `/var/www` to the root so that they are not lost when the GNS3 project is closed. All of the following scripts are called in `/root/.bashrc`.
> _Script Yudhistira_

```bash
echo nameserver 192.168.122.1 > /etc/resolv.conf

apt-get update
apt-get install bind9 -y
cp -r -f /root/dns/bind /etc/
service bind9 restart
```

> _Script Werkudara_

```bash
echo nameserver 192.168.122.1 > /etc/resolv.conf

apt-get update
apt-get install bind9 -y
cp -r -f /root/dns/bind /etc/
service bind9 restart
```

> _Script Arjuna_

```bash
echo nameserver 192.168.122.1 > /etc/resolv.conf

apt-get update
apt-get install nginx -y
service nginx start
cp -r -f /root/arjunalb/nginx /etc
service nginx restart
```

> _Script Abimanyu_

```bash
echo nameserver 192.168.122.1 > /etc/resolv.conf

cp -rf ~/webserver/www /var

apt-get update && apt-get install nginx php php-fpm apache2 libapache2-mod-php7.0 -y
service nginx start
cp -r -f /root/webserver/nginx /etc
service php7.0-fpm start
service php7.0-fpm restart
rm -rf /etc/nginx/sites-enabled/default
service nginx restart

cp -rf ~/webserver/apache2 /etc
a2ensite abimanyu.i06.conf
a2ensite parikesit.i06.conf
a2ensite rjp.baratayuda.abimanyu.i06.conf

htpasswd -bc /etc/apache2/.htpasswd Wayang baratayudad15

a2enmod authn_core authz_core authn_file authz_user rewrite

service apache2 restart
```

> _Script Prabakusuma_

```bash
echo nameserver 192.168.122.1 > /etc/resolv.conf

apt-get update && apt install nginx php php-fpm -y
service nginx start
cp -r -f /root/webserver/nginx /etc
cp -r -f /root/webserver/www /var
service php7.0-fpm start
service php7.0-fpm restart
rm -rf /etc/nginx/sites-enabled/default
service nginx restart
```

> _Script Wisanggeni_

```bash
echo nameserver 192.168.122.1 > /etc/resolv.conf

apt-get update && apt install nginx php php-fpm -y
service nginx start
cp -r -f /root/webserver/nginx /etc
cp -r -f /root/webserver/www /var
service php7.0-fpm start
service php7.0-fpm restart
rm -rf /etc/nginx/sites-enabled/default
service nginx restart
```

> _Script Nakula_

```bash
echo 'nameserver 192.231.2.2
nameserver 192.231.2.3 ' > /etc/resolv.conf

apt-get update -y
apt-get install dnsutils -y
apt-get install lynx -y
```

> _Script Sadewa_

```bash
echo 'nameserver 192.231.2.2
nameserver 192.231.2.3 ' > /etc/resolv.conf

apt-get update -y
apt-get install dnsutils -y
apt-get install lynx -y
```
