```sh
wget https://sonatype-download.global.ssl.fastly.net/repository/repositoryManager/3/nexus-3.14.0-04-unix.tar.gz
tar -xzf nexus-3.14.0-04-unix.tar.gz
mv nexus-3.14.0-04 /opt/nexus
vim /etc/systemd/system/nexus.service

####
[Unit]
Description=nexus service
After=network.target

[Service]
Type=forking
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
User=nexus
Restart=on-abort

[Install]
WantedBy=multi-user.target
####

sudo systemctl daemon-reload
sudo systemctl enable nexus.service
sudo systemctl start nexus.service
```

Maven的配置文件settings.xml：
```xml
<server>
      <id>maven-snapshots</id>
      <username>${NAME}</username>
      <password>${PASSWORD}</password>
</server>
```

项目的pom.xml:
```xml
<distributionManagement>
        <repository>
            <id>maven-releases</id>
            <name>Maven Release Repository</name>
            <url>http://${IP}:${PORT}/repository/maven-releases/</url>
        </repository>
        <snapshotRepository>
            <id>maven-snapshots</id>
            <name>Maven Snapshot Repository</name>
            <url>http://${IP}:${PORT}/repository/maven-snapshots/</url>
        </snapshotRepository>
</distributionManagement>
```
vim /usr/share/maven/conf/settings.xml
