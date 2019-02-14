# synergy-config

***

- **Compiling** [**Synergy-Core**](https://github.com/symless/synergy-core). 
  - Compiling [*wiki*](https://github.com/symless/synergy-core/wiki/Compiling) 
  - *Compile with Docker*:
    ```
    git clone --depth 1 https://github.com/symless/synergy-core.git && cd synergy-core
    docker run --rm -ti -v $PWD:/usr/src/synergy-core ubuntu:18.04 bash -c 'apt update && apt install -y qtcreator qtbase5-dev cmake make g++ xorg-dev libssl-dev libx11-dev libsodium-dev libgl1-mesa-glx libegl1-mesa libcurl4-openssl-dev libavahi-compat-libdnssd-dev qtdeclarative5-dev libqt5svg5-dev libsystemd-dev git && rm -rf /usr/src/synergy-core/build && mkdir /usr/src/synergy-core/build && cd /usr/src/synergy-core/build && cmake .. && make'
    sudo cp build/bin/synergy-core /usr/local/bin/synergy-core 
    ```
---

***
- *Configure Synergy Server*
    ```
    sudo curl -o '/etc/synergy.conf' 'https://raw.githubusercontent.com/anadion/synergy-config/master/synergy.conf'
    sudo curl -o '/lib/systemd/system/synergy-server@.service' 'https://raw.githubusercontent.com/anadion/synergy-config/master/synergy-server%40.service'
    sudo touch /var/log/synergy.log && chown ${USER}:${USER} /var/log/synergy.log
    ```
    - If you **don't** need encryption over ssh
      ```
      sudo sed -i 's/127.0.0.1:24800/:24800/g' /lib/systemd/system/synergy-server@.service
      sudo systemctl start synergy-server@${USER}
      sudo systemctl enable synergy-server@${USER}
      ```
    
- *Configure Synergy Client*
    - Copy binary file `/usr/local/bin/synergy-core` from Synergy Server to Client
    ```
    curl -o '/lib/systemd/system/synergy-client@.service' 'https://raw.githubusercontent.com/anadion/synergy-config/master/synergy-client%40.service'
    SYNERGY_SERVER_IP="Put here Synergy Server IP adress"
    SSH_USER="Put here ssh username"
    ```
    - If you **don't** need encryption over ssh
      ```
      sudo sed -i 's/localhost/${SYNERGY_SERVER_IP}/g' /lib/systemd/system/synergy-client@.service
      sudo systemctl daemon-reload
      sudo systemctl restart synergy-server@${USER}
      ```
- *SSH Tunnel*
    ```
    SSH_USER="Put here ssh username"
    ssh-copy-id ${SSH_USER}@${SYNERGY_SERVER_IP}
    # And Enter password
    
    curl -o '/lib/systemd/system/synergy-tunel@.service' 'https://raw.githubusercontent.com/anadion/synergy-config/master/synergy-tunel%40.service'
    sudo sed -i 's/SYNERGY_SERVER_IP/${SYNERGY_SERVER_IP}/g; s/SSH_USER/${SSH_USER}/g' /lib/systemd/system/synergy-client@.service   
    sudo systemctl daemon-reload 
    sudo systemctl start synergy-tunel@${USER} synergy-client@${USER}
    sudo systemctl enable synergy-client@${USER} synergy-tunel@${USER}
    ```