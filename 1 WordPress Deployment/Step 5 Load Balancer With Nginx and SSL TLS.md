## LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS

It is also extremely important to ensure that connections to our Web solutions are secure and information is encrypted in transit – we will also cover connection over secured HTTP (HTTPS protocol), its purpose and what is required to implement it.

When data is moving between a client (browser) and a Web Server over the Internet – it passes through multiple network devices and, if the data is not encrypted, it can be relatively easy intercepted by someone who has access to the intermediate equipment. This kind of information security threat is called Man-In-The-Middle (MIMT) attack.

This threat is real – users that share sensitive information (bank details, social media access credentials, etc.) via non-secured channels, risk their data to be compromised and used by cybercriminals.

SSL and its newer version, TSL – is a security technology that protects connection from MITM attacks by creating an encrypted session between browser and Web server. Here we will refer this family of cryptographic protocols as SSL/TLS – even though SSL was replaced by TLS, the term is still being widely used.

SSL/TLS uses digital certificates to identify and validate a Website. A browser reads the certificate issued by a Certificate Authority (CA) to make sure that the website is registered in the CA so it can be trusted to establish a secured connection.

## Configure Nginx as a Load Balancer

CONFIGURE NGINX AS A LOAD BALANCER we can either uninstall Apache from the existing Load Balancer server, or create a fresh installation of Linux for Nginx.

- Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx LB (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 – this port is used for secured HTTPS connections)
- Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses
- Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers
Update the instance and Install Nginx
```
sudo apt update
sudo apt install nginx
```
Configure Nginx LB using Web Servers’ names defined in /etc/hosts

Hint: Read this blog to read about /etc/host

Open the default nginx configuration file
```
sudo vi /etc/nginx/nginx.conf
```
```
#insert following configuration into http section

 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }

#comment out this line
#       include /etc/nginx/sites-enabled/*;

```
Restart Nginx and make sure the service is up and running
```
sudo systemctl restart nginx
sudo systemctl status nginx
```
## REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES
Let us make necessary configurations to make connections to our Tooling Web Solution secured!

In order to get a valid SSL certificate – we need to register a new domain name, we can do it using any Domain name registrar – a company that manages reservation of domain names. The most popular ones are: Godaddy.com, Domain.com, Bluehost.com.

- Register a new domain name with any registrar of our choice in any domain zone (e.g. .com, .net, .org, .edu, .info, .xyz or anyother)
- Assign an Elastic IP to our Nginx LB server and associate our domain name with this Elastic IP

we might have noticed, that every time we restart or stop/start our EC2 instance – we get a new public IP address. When we want to associate our domain name – it is better to have a static IP address that does not change after reboot. Elastic IP is the solution for this problem, learn how to allocate an Elastic IP and associate it with an EC2 server on this page.

- Update A record in our registrar to point to Nginx LB using Elastic IP address Learn how associate our domain name to our Elastic IP on this page.
Check that our Web Servers can be reached from our browser using new domain name using HTTP protocol – http://<our-domain-name.com>
- Configure Nginx to recognize our new domain name Update our nginx.conf with server_name www.<our-domain-name.com> instead of server_name www.domain.com
- Install certbot and request for an SSL/TLS certificate Make sure snapd service is active and running
```
  sudo systemctl status snapd
```
Install certbot
```
sudo snap install --classic certbot
```
Request our certificate (just follow the certbot instructions – we will need to choose which domain we want our certificate to be issued for, domain name will be looked up from nginx.conf file so make sure we have updated it).
```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
```
Test secured access to our Web Solution by trying to reach https://<our-domain-name.com>

we shall be able to access our website by using HTTPS protocol (that uses TCP port 443) and see a padlock pictogram in our browser’s search string.

Click on the padlock icon and we can see the details of the certificate issued for our website.
- Set up periodical renewal of our SSL/TLS certificate By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently.
we can test renewal command in dry-run mode
```
sudo certbot renew --dry-run
```
Best pracice is to have a scheduled job that to run renew command periodically. Let us configure a cronjob to run the command twice a day.

To do so, lets edit the crontab file with the following command:
```
crontab -e
```
Add following line:
```
* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
```
we can always change the interval of this cronjob if twice a day is too often by adjusting schedule expression.

we can also use this handy online cron expression editor. ( https://crontab.guru/ )

Congratulations! we have just implemented an Nginx Load Balancing Web Solution with secured HTTPS connection with periodically updated SSL/TLS certificates.
