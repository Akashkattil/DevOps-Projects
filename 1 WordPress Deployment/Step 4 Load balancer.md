When we access a website in the Internet we use an URL and we do not really know how many servers are out there serving our requests, this complexity is hidden from a regular user, but in case of websites that are being visited by millions of users per day, it is impossible to serve all the users from a single Web Server.

Each URL contains a domain name part, which is translated (resolved) to IP address of a target server that will serve requests when open a website in the Internet. Translation (resolution) of domain names is perormed by DNS servers.

When we have just one Web server and load increases – we want to serve more and more customers, we can add more CPU and RAM or completely replace the server with a more stronger one – this is called "vertical scaling". This approach has limitations – at some point we reach the maximum capacity of CPU and RAM that can be installed into server.

Another approach used to cater for increased traffic is "horizontal scaling" – distributing load across multiple Web servers. This approach is much more common and can be applied almost seamlessly and almost infinitely.

Horizontal scaling allows to adapt to current load by adding (scale out) or removing (scale in) Web servers. Adjustment of number of servers can be done manually or automatically (for example, based on some monitored metrics like CPU and Memory load).

Property of a system (in our case it is Web tier) to be able to handle growing load by adding resources, is called "Scalability".

We have  3 Web Servers and each of them had its own public IP address and public DNS name. A client has to access them by using different URLs, which is not a nice user experience to remember addresses/names of even 3 server, let alone millions of Google servers.

In order to hide all this complexity and to have a single point of access with a single public IP address/name, a Load Balancer can be used. A Load Balancer (LB) distributes clients’ requests among underlying Web Servers and makes sure that the load is distributed in an optimal way.

## Configure Apache as a Load Balancer

1, Create an Ubuntu Server 20.04 EC2 instance and name it apache-lb
2, Open TCP port 80 on apache-lb by creating an Inbound Rule in Security Group.

3, Install Apache Load Balancer on 8-apache-lb server and configure it to point traffic coming to LB to both Web Servers:
```
#Install apache2
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev

#Enable following modules:
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic

#Restart apache2 service
sudo systemctl restart apache2
```
Make sure apache2 is up and running
```
sudo systemctl status apache2
```
Configure load balancing
```
sudo vi /etc/apache2/sites-available/000-default.conf

#Add this configuration into this section <VirtualHost *:80>  </VirtualHost>

<Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/

Restart apache server

sudo systemctl restart apache2
```

4, Verify that our configuration works – try to access  LB’s public IP address or Public DNS name from  browser:
```
http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php
```
Note: If our Project we mounted /var/log/httpd/ from  Web Servers to the NFS server – unmount them and make sure that each Web Server has its own log directory.

Open two ssh/Putty consoles for both Web Servers and run following command:
```
sudo tail -f /var/log/httpd/access_log
```
Try to refresh  browser page http:///index.php several times and make sure that both servers receive HTTP GET requests from  LB – new records must appear in each server’s log file. The number of requests to each server will be approximately the same since we set loadfactor to the same value for both servers – it means that traffic will be disctributed evenly between them.

If we have configured everything correctly –  users will not even notice that their requests are served by more than one server.
