

```sh
sudo apt-get install curl gnupg -y

curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.deb.sh | sudo bash

sudo apt-get install apt-transport-https

sudo tee /etc/apt/sources.list.d/bintray.rabbitmq.list <<EOF 
## Installs the latest Erlang 22.x release.
## Change component to "erlang-21.x" to install the latest 21.x version.
## "bionic" as distribution name should work for any later Ubuntu or Debian release.
## See the release to distribution mapping table in RabbitMQ doc guides to learn more.
deb https://dl.bintray.com/rabbitmq-erlang/debian bionic erlang
deb https://dl.bintray.com/rabbitmq/debian bionic main
EOF

sudo apt-get update

sudo apt-get install rabbitmq-server -y --fix-missing

sudo rabbitmq-server start
```

[Configuration](https://www.rabbitmq.com/configure.html)

配置文件:

[rabbitmq.conf.example](https://raw.githubusercontent.com/rabbitmq/rabbitmq-server/master/docs/rabbitmq.conf.example)

```sh

sudo rabbitmqctl add_user cofcool
sudo rabbitmqctl set_permissions --vhost / cofcool '.*' '.*' '.*'
```

```sh
sudo rabbitmq-plugins enable rabbitmq_management
sudo rabbitmq-server -detached
sudo rabbitmqctl stop
```