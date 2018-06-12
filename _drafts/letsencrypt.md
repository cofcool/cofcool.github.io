# Let’s Encrypt

[certbot](https://certbot.eff.org)


```sh
wget https://dl.eff.org/certbot-auto
sudo ./path/to/certbot-auto --apache

Automating renewal

Certbot can be configured to renew your certificates automatically before they expire. Since Let's Encrypt certificates last for 90 days, it's highly advisable to take advantage of this feature. You can test automatic renewal for your certificates by running this command:

$ sudo ./path/to/certbot-auto renew --dry-run

If that appears to be working correctly, you can arrange for automatic renewal by adding a cron job or systemd timer which runs the following:

./path/to/certbot-auto renew

Note:

if you're setting up a cron or systemd job, we recommend running it twice per day (it won't do anything until your certificates are due for renewal or revoked, but running it regularly would give your site a chance of staying online in case a Let's Encrypt-initiated revocation happened for some reason). Please select a random minute within the hour for your renewal tasks.

An example cron job might look like this, which will run at noon and midnight every day:

0 0,12 * * * python -c 'import random; import time; time.sleep(random.random() * 3600)' && ./path/to/certbot-auto renew
```

```
yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm
vim /etc/yum.repos.d/epel.repo
```

./certbot-auto --apache -d example.com
