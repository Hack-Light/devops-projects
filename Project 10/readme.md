# PROJECT 10 - LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS

![project10](./images/1.png)

> You can either uninstall Apache from the existing Load Balancer server or create a fresh installation of Linux for Nginx. `sudo apt-get remove –purge apache2 apache2-utils`

OR

> Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it `Nginx LB` (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 – this port is used for secured HTTPS connections)

- Update `/etc/hosts`file for local DNS with Web Servers’ names (e.g. `Web1` and `Web2`) and their local IP addresses

- Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers

- Update the instance and Install Nginx

```bash
sudo apt update
sudo apt install nginx
```

![project10](./images/2.png)

![project10](./images/3.png)

![project10](./images/4.png)

- Configure Nginx LB using Web Servers’ names defined in /etc/hosts

> Hint: Read this blog to read about [/etc/host](https://linuxize.com/post/how-to-edit-your-hosts-file/)

- Open the default nginx configuration file `sudo vi /etc/nginx/nginx.conf`

- insert following configuration into http section

```config
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

![project10](./images/5.png)

- Restart Nginx and make sure the service is up and running

```bash
sudo systemctl restart nginx
sudo systemctl status nginx
```

![project10](./images/6.png)

> Side Self Study: Read more about HTTP load balancing methods and features supported by Nginx on this page

## REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES

Let us make necessary configurations to make connections to our Tooling Web Solution secured!

> In order to get a valid SSL certificate – you need to register a new domain name, you can do it using any Domain name registrar – a company that manages reservation of domain names. The most popular ones are: Godaddy.com, Domain.com, Bluehost.com.

> Register a new domain name with any registrar of your choice in any domain zone (e.g. .com, .net, .org, .edu, .info, .xyz or any other)

![project10](./images/7.png)

## ASSIGN AN ELASTIC IP TO YOUR NGINX LB SERVER AND ASSOCIATE YOUR DOMAIN NAME WITH THIS ELASTIC IP

> You might have noticed, that every time you restart or stop/start your EC2 instance – you get a new public IP address. When you want to associate your domain name – it is better to have a static IP address that does not change after reboot. Elastic IP is the solution for this problem, learn how to allocate an Elastic IP and associate it with an EC2 server on this page.

![project10](./images/8.png)

![project10](./images/9.png)

![project10](./images/10.png)

![project10](./images/11.png)

- Update `A` record in your registrar to point to Nginx LB using Elastic IP address

![project10](./images/12.png)

![project10](./images/13.png)

> Learn how associate your domain name to your Elastic IP on this page.

> Side Self Study: Read about different DNS record types and learn what they are used for.

- Check that your Web Servers can be reached from your browser using new domain name using HTTP protocol – `http://<your-domain-name.com>`

![project10](./images/14.png)

- Configure Nginx to recognize your new domain name

- Update your `nginx.conf` with

```config
server_name www.<your-domain-name.com>

# instead of

server_name www.domain.com
```

![project10](./images/15.png)

![project10](./images/16.png)

## INSTALL CERTBOT AND REQUEST FOR AN SSL/TLS CERTIFICATE

- Make sure snapd service is active and running `sudo systemctl status snapd`

![project10](./images/17.png)

- Install certbot `sudo snap install --classic certbot`

![project10](./images/18.png)

- Request your certificate (just follow the certbot instructions – you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from nginx.conf file so make sure you have updated it on the previous step).

```bash
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
```

![project10](./images/19.png)

- Test secured access to your Web Solution by trying to reach `https://<your-domain-name.com>`

![project10](./images/20.png)

- You shall be able to access your website by using HTTPS protocol (that uses `TCP port 443`) and see a padlock pictogram in your browser’s search string.

- Click on the padlock icon and you can see the details of the certificate issued for your website.

## SET UP PERIODICAL RENEWAL OF YOUR SSL/TLS CERTIFICATE

> By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently.

- You can test renewal command in dry-run mode `sudo certbot renew --dry-run`

- Best pracice is to have a scheduled job that to run renew command periodically. Let us configure a cronjob to run the command twice a day.

- To do so, lets edit the crontab file with the following command:

```bash
crontab -e
# Add following line:
* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
```

> You can always change the interval of this cronjob if twice a day is too often by adjusting schedule expression.

> Side Self Study: Refresh your cron configuration knowledge by watching this [video](https://youtu.be/4g1i0ylvx3A).

> You can also use this handy online cron [expression editor](https://crontab.guru/).

**Congratulations!**

_You have just implemented an Nginx Load Balancing Web Solution with secured HTTPS connection with periodically updated SSL/TLS certificates._
