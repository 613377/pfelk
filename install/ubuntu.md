## Custom Installation Guide (pfSense/OPNsense) + Elastic Stack 

## Table of Contents

- [Preparation](#preparation)
- [Installation](#installation)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)
- [Services](#services)

# Preparation

### 1. Disabling Swap
Swapping should be disabled for performance and stability.
```
sudo swapoff -a
```
```
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

### 2. Configuration Date/Time Zone
The box running this configuration will reports firewall logs based on its clock.  The command below will set the timezone to Eastern Standard Time (EST).  To view available timezones type `sudo timedatectl list-timezones`
```
sudo timedatectl set-timezone EST
```

### 3. Download and install the public GPG signing key
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
```

### 4. Download and install apt-transport-https package
```
sudo apt install apt-transport-https
```

### 5. Add Elasticsearch|Logstash|Kibana Repositories (version 8+)
```
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
```

### 6. Update
```
sudo apt update
```

### 7. Install MaxMind (Optional)
Follow the steps [here](https://github.com/pfelk/pfelk/wiki/How-To:-MaxMind-via-GeoIP-with-pfELK), to install and utilize MaxMind. Otherwise the built-in GeoIP from Elastic will be utilized.

# Installation
- Elasticsearch v8+ | Kibana v8+ | Logstash v8+

### 8. Install Elasticsearch|Kibana|Logstash
```
sudo apt install elasticsearch; sudo apt install kibana; sudo apt install logstash
```

# Configuration

### 9. Configure Kibana
```
sudo sed -i 's/#server.port: 5601/server.port: 5601/'  /etc/kibana/kibana.yml
sudo sed -i 's/#server.host: "localhost"/server.host: "0.0.0.0"/' /etc/kibana/kibana.yml
```

### 10. Configure Logstash
```
sudo wget -q -N https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/config/pipelines.yml -P /etc/logstash/
```

### 11. Create Required Directories 
```
sudo mkdir -p /etc/pfelk/{conf.d,config,logs,databases,patterns,scripts,templates}
```

### 12. (Required) Download the following configuration files
```
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/conf.d/01-inputs.pfelk -P /etc/pfelk/conf.d/
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/conf.d/02-firewall.pfelk -P /etc/pfelk/conf.d/
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/conf.d/05-apps.pfelk -P /etc/pfelk/conf.d/
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/conf.d/30-geoip.pfelk -P /etc/pfelk/conf.d/
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/conf.d/49-cleanup.pfelk -P /etc/pfelk/conf.d/
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/conf.d/50-outputs.pfelk -P /etc/pfelk/conf.d/
```

### 13. (Optional) Download the following configuration files
```
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/conf.d/20-interfaces.pfelk -P /etc/pfelk/conf.d/
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/conf.d/35-rules-desc.pfelk -P /etc/pfelk/conf.d/
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/conf.d/36-ports-desc.pfelk -P /etc/pfelk/conf.d/
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/conf.d/37-enhanced_user_agent.pfelk -P /etc/pfelk/conf.d/
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/conf.d/38-enhanced_url.pfelk -P /etc/pfelk/conf.d/
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/conf.d/45-enhanced_private.pfelk -P /etc/pfelk/conf.d/
```

### 14. Download the grok pattern
```
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/patterns/pfelk.grok -P /etc/pfelk/patterns/
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/patterns/openvpn.grok -P /etc/pfelk/patterns/
```

### 15. (Optional) Download the Database(s)
```
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/databases/private-hostnames.csv -P /etc/pfelk/databases/
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/databases/rule-names.csv -P /etc/pfelk/databases/
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/databases/service-names-port-numbers.csv -P /etc/pfelk/databases/
```

### 16. (Optional) Configure Firewall Rule Database
To configure pfSense/OPNsense to update the firewall rule database, follow [this reference](https://github.com/pfelk/pfelk/wiki/References:-Rule-Descriptions).

### 17. (Optional) Amend 20-interfaces.pfelk as desired, to map/reference the interface.name, interface.alias and network.name fields. 
Amend `interface.name`, `interface.alias` and `network.name` fields via [Wiki page](https://github.com/pfelk/pfelk/wiki/References:-Customized-Interface-Names)

# Troubleshooting
### 18. Create Logging Directory 
```
sudo mkdir -p /etc/pfelk/logs
```

### 19. Download Script
`error-data.sh`
```
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/scripts/error-data.sh -P /etc/pfelk/scripts/
```

### 20. Make Script Executable
`error-data.sh` 
```
sudo chmod +x /etc/pfelk/scripts/error-data.sh
```

# Services
### 21. Start Services on Boot (you'll need to reboot or start manually to proceed)
```
sudo /bin/systemctl daemon-reload
```
```
sudo /bin/systemctl enable elasticsearch.service
```
```
sudo /bin/systemctl enable kibana.service
```
```
sudo /bin/systemctl enable logstash.service
```

# Logstash Stop on Failure 
### 22. Amend logstash.service to Stop on Failure
```
sed -i 's?ExecStart=/usr/share/logstash/bin/logstash "--path.settings" "/etc/logstash"?ExecStart=/usr/share/logstash/bin/logstash "--pipeline.unsafe_shutdown" "--path.settings" "/etc/logstash"?' /etc/systemd/system/logstash.service
```

### 23. Start Services Manually
```
systemctl start elasticsearch.service
```
```
systemctl start kibana.service
```
```
systemctl start logstash.service
```

**[Install](ubuntu.md)** • <sub>[Security](security.md)</sub> • <sub>[Templates](templates.md)</sub> • <sub>[Configuration](configuration.md)</sub>
