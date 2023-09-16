# Securing-Apache-Server
## Objectives:
• To setup a secure web server using Apache and digital certificates.  
 
## Instruction:
In this lab, you will setup a secure web server using Apache and digital certificates. Apache is
the most widely-used web servers in the world. In fact, it is one of the most widely used open
source software in the world!  

However, the HTTP protocol has no security built into it. To handle this issue, a security
overlay has been introduced in the transport layer in 1994. It is known as Secure Sockets Layer
(SSL). Later it was transformed into Transport Layer Security. SSL/TLS utilises cryptographic
mechanisms to create a secure connection within an unprotected network such as the
Internet. Combined with the HTTP (Hypertext Transfer Protocol, the de-facto protocol for
web), the SSL/TLS introduces the notion of HTTPS (HTTP Secure).    

The process is very briefly described below. To establish an HTTPS session, a web browser
sends a request to a web server with the list of supported ciphers and hash functions by the
browser. The web server chooses a cipher from the list and sends a response back to the web
server with its choice along with a digital certificate indicating its identity. The web browser
confirms the identity by verifying the certificate. The certificate also contains a public key of
the web server. The public key is then used to encrypt a secret value, which is then sent to
the web server. The web server decrypts the secret value with its private key. Thus, a shared
secret is established between the web browser and the server. This key is then used to utilise
a symmetric encryption between the browser and the server for any subsequent
communication. Thus, one of the crucial steps in setting up a secure web server is to create a digital certificate.
In this lab, you will perform different tasks to setup a secure web server using Apache and a
digital certificate.  
## Setting up an Apache web server
The first step to setup a secure web server is to setup an Apache web server. <!--The following is
a tutorial collected from: [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-toinstall-the-apache-web-server-on-ubuntu-18-04). -->
### Step 1 — Installing Apache  
Apache is available within Ubuntu's default software repositories, making it possible to install
it using conventional package management tools.
Let's begin by updating the local package index to reflect the latest upstream changes. If apt
is not recognised as a command, try apt-get instead of apt.  
```bash
sudo apt update
```
Then, install the apache2 package:  
```bash
sudo apt install apache2
```  
After confirming the installation, apt will install Apache and all required dependencies.  
### Step 2 — Adjusting the Firewall  
Before testing Apache, it's necessary to modify the firewall settings to allow outside access to
the default web ports. Assuming that you followed the instructions in the prerequisites, you
should have a UFW firewall configured to restrict access to your server.
During installation, Apache registers itself with UFW to provide a few application profiles that
can be used to enable or disable access to Apache through the firewall.
List the ufw application profiles by typing:  
```bash
sudo ufw app list
```
You will see a list of the application profiles.    
As you can see, there are three profiles available for Apache:  
- Apache: This profile opens only port 80 (normal, unencrypted web traffic)  
- Apache Full: This profile opens both port 80 (normal, unencrypted web traffic) and
port 443 (TLS/SSL encrypted traffic)  
- Apache Secure: This profile opens only port 443 (TLS/SSL encrypted traffic)  

It is recommended that you enable the most restrictive profile that will still allow the traffic
you've configured. Since we haven't configured SSL for our server yet in this guide, we will
only need to allow traffic on port 80:  
```bash
sudo ufw allow 'Apache Full'
```
You can verify the change by typing:
```bash
sudo ufw status
```
You should see HTTP traffic allowed in the displayed output.  

### Step 3 — Checking your Web Server
At the end of the installation process, Ubuntu 18.04 starts Apache. The web server should
already be up and running.
Check with the systemd init system to make sure the service is running by typing:
```bash
sudo systemctl status apache2
```
As you can see from this output, the service appears to have started successfully. However,
the best way to test this is to request a page from Apache.
You can access the default Apache landing page to confirm that the software is running
properly through your IP address or by just typing localhost (127.0.0.1) in the browser. Let us
use pkilabserver.com as our domain name. To get our computers recognize this domain
name, let us add the following entry to /etc/hosts; this entry basically maps the domain name
pkilabserver.com to our localhost (i.e., 127.0.0.1)`127.0.0.1`or `pkilabserver.com`  
Now, to check the installation of Apache, enter this domain or its IP address into your
browser's address bar: `http://pkilabserver.com` or `http://127.0.0.1` or `http://localhost`  
You should see the default Apache web page.  
### Step 4 — Managing the Apache Process
Now that you have your web server up and running, let's go over some basic management
commands.  
To stop your web server, type:
```bash
sudo systemctl stop apache2
```
To start the web server when it is stopped, type:
```bash
sudo systemctl start apache2
```
To stop and then start the service again, type:
```bash
sudo systemctl restart apache2
```
If you are simply making configuration changes, Apache can often reload without dropping
connections. To do this, use this command:
```bash
sudo systemctl reload apache2
```
By default, Apache is configured to start automatically when the server boots. If this is not
what you want, disable this behavior by typing:
```bash
sudo systemctl disable apache2
```
To re-enable the service to start up at boot, type:
```bash
sudo systemctl enable apache2
```
Apache should now start automatically when the server boots again.  
### Step 5 — Setting Up Virtual Hosts
When using the Apache web server, you can use virtual hosts (similar to server blocks in
Nginx) to encapsulate configuration details and host more than one domain from a single
server. We will set up another domain called `example.com`.
Apache generally has one server block enabled by default that is configured to serve
documents from the /var/www/html directory. While this works well for a single site, it can
become unwieldy if you are hosting multiple sites. Instead of modifying `/var/www/html`, let's
create a directory structure within `/var/www` for our `example.com` site,
leaving `/var/www/html` in place as the default directory to be served if a client request
doesn't match any other sites.  

Create the directory for example.com as follows, using the -p flag to create any necessary
parent directories: Change `example.com` with your prefered domain name.
```bash
sudo mkdir -p /var/www/example.com/html
```
Next, assign ownership of the directory with the $USER environmental variable:
```bash
sudo chown -R $USER:$USER /var/www/example.com/html
```
The permissions of your web roots should be correct if you haven't modified
your unmask value, but you can make sure by typing:
```bash
sudo chmod -R 755 /var/www/example.com
```
You can Go to root mode using sudo(optional)
```bash
sudo su
```
Next, create a sample index.html page using nano or your favorite editor:
```bash
sudo nano /var/www/example.com/html/index.html
```
Inside, add the following sample HTML:
```bash
<html>
  <head>
     <title>Welcome to Example.com!</title>
  </head>
  <body>
     <h1>Success! The example.com server block is working!</h1>
  </body>
</html>
```
Save and close the file when you are finished.
In order for Apache to serve this content, it's necessary to create a virtual host file with the
correct directives. Instead of modifying the default configuration file located
at `/etc/apache2/sites-available/000-default.conf` directly, let's make a new one
at `/etc/apache2/sites-available/example.com.conf`:
```bash
sudo nano /etc/apache2/sites-available/example.com.conf
```
Paste in the following configuration block, which is similar to the default, but updated for our
new directory and domain name:
```bash
<VirtualHost *:80>
    ServerAdmin admin@example.com
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example.com/html
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Notice that we've updated the DocumentRoot to our new directory and ServerAdmin to an
email that the example.com site administrator can access. We've also added two
directives: ServerName, which establishes the base domain that should match for this virtual
host definition, and ServerAlias, which defines further names that should match as if they
were the base name.  

Save and close the file when you are finished.  

Go to `/etc/hosts` and add the following code
```bash
sudo nano /etc/hosts
```
add this:
```bash
127.0.0.1    example.com
```
Let's enable the file with the a2ensite tool:
```bash
sudo a2ensite example.com.conf
```
Disable the default site defined in 000-default.conf:
```bash
sudo a2dissite 000-default.conf
```
Next, let's test for configuration errors:
```bash
sudo apache2ctl configtest
```
If you see a “Syntax OK” output, then it’s properly configured.
Restart Apache to implement your changes:
```bash
sudo systemctl restart apache2
```
Apache should now be serving your domain name. You can test this by navigating
to `http://example.com`, where you should see something like this:  
```bash
Success! The example.con virtual host is working!
```
### Other steps – Getting Familiar with Important Apache Files and Directories
Now that you know how to manage the Apache service itself, you should take a few minutes
to familiarize yourself with a few important directories and files.  
### Content
- /var/www/html: The actual web content, which by default only consists of the default
Apache page you saw earlier, is served out of the /var/www/html directory. This can
be changed by altering Apache configuration files.  
### Server Configuration
- /etc/apache2: The Apache configuration directory. All of the Apache configuration
files reside here.  
- /etc/apache2/apache2.conf: The main Apache configuration file. This can be modified
to make changes to the Apache global configuration. This file is responsible for loading
many of the other files in the configuration directory.  
- /etc/apache2/ports.conf: This file specifies the ports that Apache will listen on. By
default, Apache listens on port 80 and additionally listens on port 443 when a module
providing SSL capabilities is enabled.  
- /etc/apache2/sites-available/: The directory where per-site virtual hosts can be
stored. Apache will not use the configuration files found in this directory unless they
are linked to the sites-enabled directory. Typically, all server block configuration is
done in this directory, and then enabled by linking to the other directory with
the a2ensite command.  
- /etc/apache2/sites-enabled/: The directory where enabled per-site virtual hosts are
stored. Typically, these are created by linking to configuration files found in the sitesavailabledirectory with the a2ensite. Apache reads the configuration files and links
found in this directory when it starts or reloads to compile a complete configuration.  
- /etc/apache2/conf-available/, /etc/apache2/conf-enabled/: These directories have
the same relationship as the sites-available and sites-enabled directories, but are
used to store configuration fragments that do not belong in a virtual host. Files in
the conf-available directory can be enabled with the a2enconf command and disabled
with the a2disconf command.  
- /etc/apache2/mods-available/, /etc/apache2/mods-enabled/: These directories
contain the available and enabled modules, respectively. Files in ending
in .load contain fragments to load specific modules, while files ending in .conf contain
the configuration for those modules. Modules can be enabled and disabled using
the a2enmod and a2dismod command.  
### Server Logs
- /var/log/apache2/access.log: By default, every request to your web server is recorded
in this log file unless Apache is configured to do otherwise.  
- /var/log/apache2/error.log: By default, all errors are recorded in this file.
The LogLeveldirective in the Apache configuration specifies how much detail the error
logs will contain.  

## Becoming a certificate authority
A Certificate Authority (CA) is a trusted entity that issues digital certificates. The digital
certificate certifies the ownership of a public key by the named subject of the certificate. A
number of commercial CAs are treated as root CAs; VeriSign is the largest CA at the time of
writing. Users who want to get digital certificates issued by the commercial CAs need to pay
those CAs.
In this lab, we need to create digital certificates, but we are not going to pay any commercial
CA. We will become a root CA ourselves, and then use this CA to issue certificate for others
(e.g. servers). In this task, we will make ourselves a root CA, and generate a certificate for this
CA. Unlike other certificates, which are usually signed by another CA, the root CA’s certificates
are self-signed. Root CA’s certificates are usually pre-loaded into most operating systems,
web browsers, and other software that rely on PKI. Root CA’s certificates are unconditionally
trusted.  
***The Configuration File openssl.conf***: In order to use OpenSSL to create certificates, you have
to have a configuration file. The configuration file usually has an extension .cnf. It is used by
three OpenSSL commands: ca, req and x509. The manual page of openssl.conf can be found
using Google search. You can also get a copy of the configuration file from
`/usr/lib/ssl/openssl.cnf`. After copying this file into your current directory, you need to create
several sub-directories as specified in the configuration file (look at the [CA default] section)

`dir` = `./demoCA` where everything is kept  
`certs` = `$dir/certs` where the issued certs are kept   
`crl_dir` = `$dir/crl` where the issued crl are kept  
`new_certs_dir` = `$dir/newcerts` default place for new certs  
`database` = `$dir/index.txt` database index file  
`serial` = `$dir/serial` the current serial file   

```bash
cd /var/www/example.com/
```
Create demoCA directory
```bash
mkdir demoCA
```
Now create other directories.
```bash
cd demoCA/
```
```bash
mkdir certs crl newcerts
```
Now create database index and serial file
```bash
touch index.txt serial
```
Put a rangom number on the serial file(e.g. 1000)
```bash
nano serial
```
Save and Close the file.  

Now copy the `openssl.cnf` file from `/usr/lib/ssl` to your `/var/www/example.com/demoCA/` directory.
Go to `/usr/lib/ssl` directory using command line or GUI.
```bash
sudo cp openssl.cnf /var/www/example.com/demoCA/
```

### Certificate Authority (CA)
As we described before, we need to generate a self-signed
certificate for our CA. This means that this CA is totally trusted, and its certificate will serve as
the root certificate. You can run the following command to generate the self-signed certificate
for the CA:
```bash
openssl req -new -x509 -keyout ca.key -out ca.crt -config openssl.cnf
```
You will be prompted for information and a password. Do not lose this password, because
you will have to type the passphrase each time you want to use this CA to sign certificates for
others. You will also be asked to fill in some information, such as the Country Name, Common
Name, etc. The output of the command are stored in two files: ca.key and ca.crt. The file
ca.key contains the CA’s private key, while ca.crt contains the public-key certificate.
### Creating a certificate for example.com
After becoming a root CA, we are ready to sign digital certificates for our customers. Our first
customer is a company called example.com. For this company to get a digital certificate from
a CA, it needs to go through three steps.  
- Step 1: **Generate public/private key pair**: The company needs to first create its own
public/private key pair. We can run the following command to generate an RSA key pair (both
private and public keys). You will also be required to provide a password to protect the keys.
The keys will be stored in the file server.key:  
```bash
openssl genrsa -des3 -out server.key 2048
```
- Step 2: **Generate a Certificate Signing Request (CSR)**: Once the company has the key file, it
should generates a Certificate Signing Request (CSR). The CSR will be sent to the CA, who will
generate a certificate for the key (usually after ensuring that identity information in the CSR
matches with the server’s true identity). Please use example.com as the common name of
the certificate request.
```bash
openssl req -new -key server.key -out server.csr -config openssl.cnf
```
- Step 3: **Generating Certificates**: The CSR file needs to have the CA’s signature to form a
certificate. In the real world, the CSR files are usually sent to a trusted CA for their signature.
In this lab, we will use our own trusted CA to generate certificates:
At first go to immediate directory from demoCA.
```bash
cd ..
```
```bash
openssl ca -in ./demoCA/server.csr -out ./demoCA/server.crt -cert ./demoCA/ca.crt -keyfile ./demoCA/ca.key -config ./demoCA/openssl.cnf
```
If OpenSSL refuses to generate certificates, it is very likely that the names in your requests do
not match with those of CA. Fix this and re-issue the above command.
### Using OpenSSL to demonstrate HTTPS
In this lab, we will explore how public-key certificates are used by web sites to secure web
browsing. Next, let us launch a simple web server with the certificate generated in the
previous task. OpenSSL allows us to start a simple web server using the s_server command.
Use the following steps:  
- Step 1: Combine the secret key and certificate into one file
```bash
cd demoCA
```
```bash
cp server.key server.pem
```
```bash
cat server.crt >> server.pem
```
- Step 2: Launch the web server using server.pem
```bash
openssl s_server -cert server.pem -www
```
For multiple server use -accept option
```bash
openssl s_server -cert server.pem -www -accept port_number
```
By default, the server will listen on port 4433. You can alter that using the `-accept` option.
Now, you can access the server using the following URL: `https://example.com:4433/`. Most
likely, you will get an error message from the browser. In Firefox, you will see a message like
the following: “example.com:4433 uses an invalid security certificate. The certificate is not
trusted because the issuer certificate is unknown”.  

Had this certificate been assigned by VeriSign, we will not have such an error message,
because VeriSign’s certificate is very likely preloaded into Firefox’s certificate repository
already. Unfortunately, the certificate of example.com is signed by our own CA (i.e., using
ca.crt), and this CA is not recognized by Firefox. There are two ways to get Firefox to accept
our CA’s self-signed certificate.  

We can request Mozilla to include our CA’s certificate in its Firefox software, so everybody
using Firefox can recognize our CA. This is how the real CAs, such as VeriSign, get their
certificates into Firefox. Unfortunately, our own CA does not have a large enough market for
Mozilla to include our certificate, so we will not pursue this direction.  


**Load ca.crt into Firefox**: We can manually add our CA’s certificate to the Firefox browser by
clicking the following menu sequence:
- `Settings -> Privacy and Security -> Security -> View Certificates -> Authorities -> Import your ca.crt file`  
You will see a list of certificates that are already accepted by Firefox. From here, we can
“import” our own certificate. Please import ca.crt, and select the following option: “Trust this
CA to identify web sites”. You will see that our CA’s certificate is now in Firefox’s list of the
accepted certificates. Now, point the browser to` https://example.com:4433`.  

## Deploy HTTPS into Apache
Now, we will deploy the HTTPS capability into Apache web server. At first, stop the Openssl
webserver launched in the previous task. Now add the following lines into the example
configuration file: 
```bash 
sudo nano /etc/apache2/sites-available/example.com.conf
```
Copy and Paste the followng code. Don't remove previous code. Just paste the following code after it.
```bash
<IfModule mod_ssl.c>
<VirtualHost *:443>
    ServerAdmin admin@example.com
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example.com/html
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
SSLEngine on
SSLCertificateFile /var/www/example.com/demoCA/server.crt 
SSLCertificateKeyFile /var/www/example.com/demoCA/server.key
</VirtualHost>
</IfModule>
```
Now, we will need to enable the ssl module in Apache which might not be enabled by default.  
Use the following command to enable the ssl module Apache.  
```bash
sudo a2enmod ssl
```
Next, use the following command to test the configuration.  
```bash
sudo apache2ctl configtest
```
If a syntax is displayed onto the terminal, it indicates everything is okay.  

Next restart the apache server using the restart command shown above.
```bash
sudo systemctl restart apache2
```
Now, try to access the `https://example.com`. If everything is properly configured, you should
be able to view the webpage in HTTPS.
If your browser is Firefox and it shows a warning, you can fix it by importing the CA certificate
as described previously. If you use Chrome and it shows a similar warning, you can also import
the CA certificate from the Manage certificate option under the Advanced setting in Chrome.  


