## Exploitation

This installation requires VMware or VirtualBox Hypervisors to be installed.

Download Ubuntu 12.04.5 installation ISO on the computer from the following website and set it up through your preferred hypervisor :

```sh
https://old-releases.ubuntu.com/releases/precise/
```
Install the Nginx web server which will be used to launch a test website that has the heartbleed vulnerability, using the following command :

```sh
sudo apt-get install nginx
```

Check and confirm that the OpenSSL version is 1.0.1 by typing in the following command :
```sh
openssl version -a
```

After confirming the version of OpenSSL, proceed to type the following command to generate a certificate : 
```sh
openssl req -new -x509 -days 365 -sha1 -newkey rsa:1024 \
-nodes -keyout heartbleed.key -out heartbleed.crt \
-subj '/O=YourCompany/OU=YourDepartment/CN=www.yoursite.com'
```

Configure the default nginx configuration file that is located in /etc/nginx/sites-available/default by pasting the following configuration (Be sure to change the path to your certificate and key) [^1] :
```sh
server {
  listen 443;
  server_name heartbleed;

  root /usr/share/nginx/www;
  index index.html index.htm;

  ## SSL
  ssl on;
  ssl_certificate /path/to/your/heartbleed.crt;
  ssl_certificate_key /path/to/your/heartbleed.key;

  ## SSL caching/optimization
  ssl_protocols        SSLv3 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers RC4:HIGH:!aNULL:!MD5;
  ssl_prefer_server_ciphers on;
  keepalive_timeout    60;
  ssl_session_cache    shared:SSL:10m;
  ssl_session_timeout  10m;

  ## SSL log files
  access_log /var/log/nginx/heartbleed_access.log;
  error_log /var/log/nginx/heartbleed/ssl_error.log;

  location / {
    proxy_set_header        Accept-Encoding   "";
    proxy_set_header        Host              $http_host;
    proxy_set_header        X-Forwarded-By    $server_addr:$server_port;
    proxy_set_header        X-Forwarded-For   $remote_addr;
    proxy_set_header        X-Forwarded-Proto $scheme;
    proxy_set_header        X-Real-IP         $remote_addr;
    proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
  }
}
``` 


With your default file configured, it's time to configure the index page by pasting the following simple html code in the root directory of the web server, by default it should be in */usr/share/nginx/www* :
```html
<html>
<head>
<title>Heartbleed Vulnerability Test</title>
<script>
document.cookie="heartbleed";
</script>
</head>
<body>
<br /><br />
<form name="input" action="index.html" method="get">
Username: <input type="text" name="username">
<br />
Password: <input type="password" name="password">
<br /><br />
<input type="submit" value="Submit">
</form>
</body>
</html>
```

Proceed to download the following python script that will run the heartbleed vulnerability from this github repository : 
#### *[exploit.py](exploit.py)* [^2]

To run the script, open a terminal in its directory or navigate to it using an already opened terminal and type the following command followed by the web server IP : 
```sh
./heartbleed-poc.py <web_server_ip_here>
```

## Patching It Up (**Quick** Method)

Before starting this method, we need to lay the **Pros** and **Cons** of using it :

| Pros | Cons |
| ------ | ------ |
| It will patch the heartbleed vulnerability by upgrading OpenSSL | It might reset some software to its default settings |
| It will make your machine more robust and up to date | OpenSSL stays at version 1.0.1 eventhough heartbleed is patched |

#### Lets Get Started
Type the following command to download and update the outdated packages with new ones :
```sh
sudo apt-get upgrade
```
Afterwards, to confirm the package updates, type the following command :
```sh
sudo apt-get update
```
```
Restart your virtual machine.
```

Run the script again as shown in the [Exploitation](#Exploitation) section to test and see if heartbleed is patched, the following text should be outputted after running the script :
```sh
Connecting...
Sending Client Hello...
Waiting for Server Hello...
Received message: type = 22, ver = 0302, length = 74
Received message: type = 22, ver = 0302, length = 638
Received message: type = 22, ver = 0302, length = 203
Received message: type = 22, ver = 0302, length = 4
Sending heartbeat request...
Unexpected EOF receiving record header - server closed connection
No heartbeat response received, server likely not vulnerable
```
## Patching It Up (**Alternate** Method)
In this method, we will be updating the version of OpenSSL to its latest publicly released version. By the time this repository is released, OpenSSL Version 3.0.6 is its latest.
Before starting with this method, we will list down the pro and con of using this method in comparison with the **quick** one.
| Pro | Con |
| ------ | ------ |
| It allows for the most up to date patch against all the latest patched vulnerabilities | It is more time consuming than the previous method |
| It does not disturb any other packages |
#### Lets Get Started
To make things easier, we will be running the linux terminal with root access using the following command : 
```sh
sudo su
```
Navigate to the following website and copy the link to the latest source code :
```
http://artfiles.org/openssl.org/source/
```
The link should look something like this :
```
http://artfiles.org/openssl.org/source/openssl-3.0.6.tar.gz
```

With the link still in the clipboard, type the following command in the terminal :
```sh
wget http://artfiles.org/openssl.org/source/openssl-3.0.6.tar.gz
```
Next, we will extract the downloaded tar gz file :
```sh
tar xzvf openssl-3.0.6.tar.gz
```
We will navigate to the directory of the extracted folder :
```sh
cd openssl-3.0.6
```
Run the config file :
```sh
./config
```
Make file and start the installation
```sh
make
```
```sh
make install
```
Type the following command to make path for the new OpenSSL version : 
```sh
ldconfig /usr/local/lib64/
```
Type the following to check the version of OpenSSL and ensure it has been updated :
```sh
openssl version -a
```
### Congratulations! Your machine is no longer vulnerable to heartbleed!
## License

[^1]: [1] Andy, “War room,” War Room, 22-May-2015. [Online]. Available: https://warroom.rsmus.com/building-a-vulnerable-box-heartbleed/. [Accessed: 12-Oct-2022].
[^2]: [2] Sensepost, “Sensepost/heartbleed-POC: Test for SSL heartbeat vulnerability (CVE-2014-0160),” GitHub. [Online]. Available: https://github.com/sensepost/heartbleed-poc. [Accessed: 12-Oct-2022].

MIT
