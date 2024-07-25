
# pfelkf requirements available in nixpkgs

previous assessment may actually be incorrect. According to pfelk/etc/pfelk/scripts/pfelk-installer.sh the dependencies are GeoIP 7.0.1 and Elasticstack 8.14.1
I suspect that kibana and logstahs

- maxmind geoip
need more research, there are packages but I'm not sure which to use
According to install/install.md this is optional. if not installed, the build in GeoIP from Elastic will be used. perhaps add support for this as a second stage for nixos and allow it to be enabled as an option.

- elasticsearch
  environment.systemPackages = [
    pkgs.elasticsearch
  ];

- logstash
 environment.systemPackages = [
    pkgs.logstash
  ];

- kibana
not found

# June 27, 2024 install step log

This logs the actions take installing pfelk on arch-ghost using docker by following /install/docker.md

## Prep

1. swap is already disabled on ghost
2. n/a
3. `sudo sysctl -w vm.max_map_count=262144`   apparently an elasticsearch req

## Installation

4. docker already installed
5. docker-compose already installed
    said it was but it wasn't
      `pacman -Syu docker-compose` to install 2.27.1-1
6. create req'd dirs `sudo mkdir -p /etc/pfelk/{conf.d,config,logs,databases,docker,patterns,scripts,templates}`
7. d/l req'd docker files

    ```bash
    sudo wget -q https://raw.githubusercontent.com/pfelk/pfelk/main/.env -P /etc/pfelk/docker/
    sudo wget -q https://raw.githubusercontent.com/pfelk/pfelk/main/docker-compose.yml -P /etc/pfelk/docker/
    sudo wget -q https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/config/logstash.yml -P /etc/pfelk/config/
    sudo wget -q https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/config/pipelines.yml -P /etc/pfelk/config/
    ```

8. d/l req'd config files

```bash
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/conf.d/01-inputs.pfelk -P /etc/pfelk/conf.d/
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/conf.d/02-firewall.pfelk -P /etc/pfelk/conf.d/
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/conf.d/05-apps.pfelk -P /etc/pfelk/conf.d/
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/conf.d/30-geoip.pfelk -P /etc/pfelk/conf.d/
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/conf.d/49-cleanup.pfelk -P /etc/pfelk/conf.d/
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/conf.d/50-outputs.pfelk -P /etc/pfelk/conf.d/
```

9. n/a
10. d/l grok patterns

```bash
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/patterns/pfelk.grok -P /etc/pfelk/patterns/
sudo wget https://raw.githubusercontent.com/pfelk/pfelk/main/etc/pfelk/patterns/openvpn.grok -P /etc/pfelk/patterns/
```

11. n/a
12. n/a
13. n/a

### Configuration

14. Configure Credentials | .env
```
sudo nvim /etc/pfelk/docker/.env
```

Amend `.env` Credentials as Desired
```
ELASTIC_PASSWORD=changeme
KIBANA_PASSWORD=changeme
LOGSTASH_PASSWORD=changeme
LICENSE=basic
```

15. Configure Credentials | 50-outputs.conf
```
sudo nvim /etc/pfelk/conf.d/50-outputs.pfelk
```

Amend 50-outputs.conf Credentials as Desired
```
    cacert => '/usr/share/logstash/config/certs/ca/ca.crt'
    user => "elastic"
    # cacert => '/etc/logstash/config/certs/http_ca.crt' #[Disable if using Docker]
    # user => "pfelk_logstash" #[Disable if using Docker]
    password => "changeme" 
```

### Services

16. done
17. `sudo docker-compose up` docker compose not installed apparently

`sudo pacman -Syu docker-compose`, installed 1.27.1-1

running up again barfs because "Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?"

Also, `docker-compose.yml` was using obsolete `version: foo`, which is not longer used as of 1.27, go figure. line was removed from file

`systemctl start docker`

`sudo docker-compose up`

container running in cli

### Accessing pfelk

1. Edited /etc/pfelk/config/logstash.yml

    ```diff
    - http.host: "0.0.0.0"
    + http.host: "127.0.0.1"
    ```

2. Rerun `sudo docker-compose up`
3. In browser visit "127.0.0.1:5601" to access pfelk dashboard
4. Next set up Templates according to install/templates.md
  "
    1. In your web browser navigate to the pfELK IP address using port 5601 (example: 192.168.0.1:5601)
    2. Click ☰ in the upper left corner
    3. Click on Dev Tools located near the bottom under the Management heading
    4. Paste the contents of each template file located in the template 📁 and links below - Component Templates - 🔺 NOTE Component Templates must be installed first and in sequential order (e.g. pfelk-settings, pfelk-mappings)
      component_pfelk-mappings - Install First
      component_pfelk-settings - Install Second
      ilm-pfelk - Install Third
      - Index Templates
      Click the green triangle after pasting the contents (one at a time) into the console
      - pfelk-firewall
  "
  I pasted these one after the other so that the console on the left contains all of the stuff in order, but I sent each request as I entered it.
  Also I stopped at pfelk-firewall because the others were optional or deprecated.
5. Now follow Dashboard Manual Method steps in install/templates.md
    imported src/pfelk/etc/pfelk/dahboard/23.09-firewall.ndjson

    skipped others because optional
6. Skipping the `Start logstash` step of install/templates.md   because I think it should already be running with the docker image
7. following pfSense steps of install/configuration.md

    1. Navigate to Status -> System Logs, then click on Settings
    2. At the bottom check Enable Remote Logging
    3. (Optional) Select a specific interface to use for forwarding
        left this as default (any)
    4. Input the ELK IP address into the field Remote log servers followed by port 5140 (i.e. I did 10.13.37.2:5140)
    5. Under Remote Syslog Contents check Everything
          NOTE!!! this maybe will cause issues if the other templates and dashboards weren't installed/????
    6. Click Save


this worked! can access at 127.0.0.1:5601





ALSO, worth noting the following from the docker-compose up output:
```
pfelk-kibana    | Kibana is currently running with legacy OpenSSL providers enabled! For details and instructions on how to disable see https://www.elastic.co/guide/en/kibana/8.14/production.html#openssl-legacy-provider
pfelk-kibana    | {"log.level":"info","@timestamp":"2024-06-27T22:52:53.990Z","log.logger":"elastic-apm-node","ecs.version":"8.10.0","agentVersion":"4.5.0","env":{"pid":7,"proctitle":"/usr/share/kibana/bin/../node/bin/node","os":"linux 6.7.7-1-MANJARO","arch":"x64","host":"1aa10ea05e9f","timezone":"UTC+00","runtime":"Node.js v20.13.1"},"config":{"active":{"source":"start","value":true},"breakdownMetrics":{"source":"start","value":false},"captureBody":{"source":"start","value":"off","commonName":"capture_body"},"captureHeaders":{"source":"start","value":false},"centralConfig":{"source":"start","value":false},"contextPropagationOnly":{"source":"start","value":true},"environment":{"source":"start","value":"production"},"globalLabels":{"source":"start","value":[["kibana_uuid","ee69b7f3-901b-4c65-a37e-1b08930d1519"],["git_rev","afbd904e868f2a48a2bbeb8ff20baee8d4aeb468"]],"sourceValue":{"kibana_uuid":"ee69b7f3-901b-4c65-a37e-1b08930d1519","git_rev":"afbd904e868f2a48a2bbeb8ff20baee8d4aeb468"}},"logLevel":{"source":"default","value":"info","commonName":"log_level"},"metricsInterval":{"source":"start","value":120,"sourceValue":"120s"},"serverUrl":{"source":"start","value":"https://kibana-cloud-apm.apm.us-east-1.aws.found.io/","commonName":"server_url"},"transactionSampleRate":{"source":"start","value":0.1,"commonName":"transaction_sample_rate"},"captureSpanStackTraces":{"source":"start","sourceValue":false},"secretToken":{"source":"start","value":"[REDACTED]","commonName":"secret_token"},"serviceName":{"source":"start","value":"kibana","commonName":"service_name"},"serviceVersion":{"source":"start","value":"8.14.1","commonName":"service_version"}},"activationMethod":"require","message":"Elastic APM Node.js Agent v4.5.0"}
```