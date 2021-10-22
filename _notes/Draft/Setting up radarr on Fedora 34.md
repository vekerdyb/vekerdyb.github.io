```  
USER=radarr  
GROUP=downloaders  
ARCHITECTURE=x64  
  
dnf install curl mediainfo sqlite wget  
mkdir /var/lib/radarr  
chown $USER:$GROUP -R /var/lib/radarr  
  
wget -O '/tmp/radarr.tar.gz' "https://radarr.servarr.com/v1/update/master/updatefile?os=linux&runtime=netcore&arch=$ARCHITECTURE"  
  
tar -xvzf /tmp/radarr.tar.gz -C /opt/  
  
groupadd $USER  
adduser -g $USER -m -c "Ronarr User" -s /sbin/nologin $USER  
usermod -aG $GROUP $USER  
  
chown $USER:$GROUP -R /opt/Radarr  
  
cat > /etc/systemd/system/radarr.service << EOF  
[Unit]  
Description=Radarr Daemon  
After=syslog.target network.target  
  
[Service]  
User=radarr  
Group=downloaders  
Type=simple  
ExecStart=/opt/Radarr/Radarr -nobrowser  
TimeoutStopSpec=20  
  
[Install]  
WantedBy=multi-user.target  
EOF  
  
systemctl enable radarr.service  
  
cat > /etc/firewalld/services/radarr.xml << EOF  
<?xml version="1.0" encoding="utf-8"?>  
<service>  
 <short>sonarr</short>  
 <description>Radarr Download Service</description>  
 <port protocol="tcp" port="7878"/>  
</service>  
EOF  
  
firewall-cmd --permanent --add-service radarr  
firewall-cmd --reload  
```  
  
  
  
Fish version:  
```  
set USER radarr  
set GROUP downloaders  
set ARCHITECTURE x64  
 
dnf install curl mediainfo sqlite wget  

groupadd $USER  
adduser -g $USER -m -c "Ronarr User" -s /sbin/nologin $USER  
usermod -aG $GROUP $USER  

mkdir /var/lib/radarr  
chown $USER:$GROUP -R /var/lib/radarr  
  
wget -O '/tmp/radarr.tar.gz' "https://radarr.servarr.com/v1/update/master/updatefile?os=linux&runtime=netcore&arch=$ARCHITECTURE"  
  
tar -xvzf /tmp/radarr.tar.gz -C /opt/  
    
chown $USER:$GROUP -R /opt/Radarr  
  
echo "[Unit]  
Description=Radarr Daemon  
After=syslog.target network.target  
  
[Service]  
User=radarr  
Group=downloaders  
Type=simple  
ExecStart=/opt/Radarr/Radarr -nobrowser  
TimeoutStopSpec=20  
  
[Install]  
WantedBy=multi-user.target" > /etc/systemd/system/radarr.service  
  
systemctl enable radarr.service  
  
systemctl start radarr.service  
systemctl status radarr.service  
  
echo "<?xml version=\"1.0\" encoding=\"utf-8\"?>    
<service>    
 <short>radarr</short>    
 <description>Radarr Download Service</description>    
 <port protocol=\"tcp\" port=\"7878\"/>    
</service>" > /etc/firewalld/services/radarr.xml  
  
firewall-cmd --permanent --add-service radarr  
firewall-cmd --reload  
  
```