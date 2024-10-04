# Installation Icecast KH & SSL

## Icecast KH Installation

Updaten des Systems

```
apt update && apt upgrade -y && apt dist-upgrade -y
```

Abh&auml;ngigkeiten installieren

```
apt install git wget build-essential libvorbis-dev libxml2 libssl-dev curl -y
```
Installation von libxslt1-dev AMD64 / x86

```
wget http://ftp.de.debian.org/debian/pool/main/libx/libxslt/libxslt1-dev_1.1.35-1_amd64.deb
apt install ./libxslt1-dev_1.1.35-1_amd64.deb
```

Installation von libxslt1-dev ARM64

```
wget http://ftp.de.debian.org/debian/pool/main/libx/libxslt/libxslt1-dev_1.1.35-1_arm64.deb
apt install ./libxslt1-dev_1.1.35-1_arm64.deb
```

Download und bereitstellung von Icecast KH

```
wget https://github.com/karlheyes/icecast-kh/archive/refs/tags/icecast-2.4.0-kh22.tar.gz
tar -xvf icecast-*.tar.gz
cd icecast-kh-icecast-2.4.0-kh22
```

Compilieren und installieren von Icecast KH

```
./configure --with-openssl
make
make install
```

Anlegen und kopieren der Konfigurationsdatei

```
mkdir -p /etc/icecast-kh
cp /usr/local/share/icecast/doc/icecast_minimal.xml.dist /etc/icecast-kh/icecast.xml
```

Anpassen der Konfigurationsdatei

```
nano /etc/icecast-kh/icecast.xml
```

Inhalt der Konfigurationsdatei

```
<icecast>
    <limits>
        <sources>2</sources>
    </limits>
    <authentication>
        <source-password>hackme1</source-password>
        <relay-password>hackme2</relay-password>
        <admin-user>admin</admin-user>
        <admin-password>hackme3</admin-password>
    </authentication>
    <directory>
        <yp-url-timeout>15</yp-url-timeout>
        <yp-url>http://dir.xiph.org/cgi-bin/yp-cgi</yp-url>
    </directory>
    <hostname>localhost</hostname>
    <listen-socket>
        <port>8000</port>
    </listen-socket>
    <fileserve>1</fileserve>
    <paths>
        <logdir>/var/log/icecast-kh</logdir>
        <webroot>/usr/local/share/icecast/web</webroot>
        <adminroot>/usr/local/share/icecast/admin</adminroot>
        <alias source="/" dest="/index.html"/>
    </paths>
    <logging>
        <accesslog>access.log</accesslog>
        <errorlog>error.log</errorlog>
      	<loglevel>3</loglevel> <!-- 4 Debug, 3 Info, 2 Warn, 1 Error -->
    </logging>
</icecast>
```

Anpassen der Sources f&uuml;r die maximalen Mountpoints

Passw&ouml;rter sollten ge&auml;ndert werden

Anlegen des Icecast-Users

```
adduser icecast --disabled-login
```

Log-Ordner anlegen und Berechtigungen anpassen

```
mkdir -p /var/log/icecast-kh
chown -R icecast:icecast /var/log/icecast-kh/
```

Installieren des Deamon Scripts

```
wget https://gist.githubusercontent.com/ssamjh/500aa158cb77a8eff79c8d1763eef339/raw/fb0a4a2d39a87affb46fc598e07da668670188fd/icecast-kh -O /etc/init.d/icecast-kh
chmod 777 /etc/init.d/icecast-kh
update-rc.d icecast-kh defaults
```

Starten und Aktivieren des Icecast Servers

```
systemctl start icecast-kh
systemctl enable icecast-kh
```

Installieren des Let's Encrypt Certbots

```
apt install certbot
```

Zertifikat anlegen

```
certbot certonly --webroot-path="/root/icecast-kh-icecast-2.4.0-kh22/web" -d 'kompletteurl'
```

Zusammenfassen von Zertifikat und Key f&uuml;r den Icecast sowie Berechtigungen vergeben

```
cat /etc/letsencrypt/live/stream.example.com/fullchain.pem /etc/letsencrypt/live/stream1.example.com/privkey.pem > /etc/icecast-kh/bundle.pem
chmod 666 /etc/icecast2/bundle.pem
```

Bearbeiten des Scripts von Lets Encrypt f&uuml;r das erneute erstellen des SSL Zertifikats f&uuml;r Icecast

```
nano /etc/letsencrypt/renewal/stream.example.com.conf
```
Folgendes in den Bereich `[renewalparams]` hinzuf&uuml;gen

```
post_hook = cat /etc/letsencrypt/live/stream.example.com/fullchain.pem /etc/letsencrypt/live/stream.example/privkey.pem > /etc/icecast-kh/bundle.pem && service icecast2 restart
```

Zum Validieren ob der Prozess richtig abl&auml;uft

```
certbot renew --dry-run
```

Erneutes Bearbeiten der Konfigurationsdatei

```
nano /etc/icecast-kh/icecast.xml
```
Folgendes in den `<paths></paths>` Bereich einsetzen

```
<ssl-certificate>/etc/icecast-kh/bundle.pem</ssl-certificate>
```
Folgendes nun in den Bereich wo bereits der andere Listen-Bereich liegt einf&uuml;gen

```
<listen-socket>
    <port>443</port>
    <ssl>1</ssl>
</listen-socket>
```

Nun Icecast neustarten

```
sudo service icecast2 restart
```

Nun sollte alles funktionieren
