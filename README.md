# pi_home
Repository for settings related to home Raspberry Pi: Home Assistant, VPN and more.

# Home Assistant

## Installing Home Assistant Core

Assuming an operating system is already setup (e.g. Ubuntu 24.04), the guide in section "Install Home Assistant Core" at the [Home Assistant webpage](https://www.home-assistant.io/installation/linux#install-home-assistant-core) was followed, with few adjustments:

1. Install dependencies:
```sh
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install -y python3 python3-dev python3-venv python3-pip bluez libffi-dev libssl-dev libjpeg-dev zlib1g-dev autoconf build-essential libopenjp2-7 libtiff6 libturbojpeg0-dev tzdata ffmpeg liblapack3 liblapack-dev libatlas-base-dev
```

2. Create a specific user (`homeassistant`) to run home assistant, set a password and enable the new user to use `sudo`:
```sh
sudo useradd -rm homeassistant
sudo passwd homeassistant # you will be prompted to set the password
sudo usermod -aG sudo homeassistant
```

3. Install `python3.13`, which is necessary for version `2025.3.1` of home assistant:
```sh
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.13 python3.13-venv python3.13-dev
```

4. Create a directory for the installation of home assistant and give permissions to the `homeassistant` user:
```sh
sudo mkdir /srv/homeassistant
sudo chown homeassistant:homeassistant /srv/homeassistant
```
and create a virtual environment for it:
```sh
sudo -u homeassistant -H -s
cd /srv/homeassistant
python3.13 -m venv .
source bin/activate
```

5. Given the virtual environment is activated, install `wheel` and `homeassistant`:
```sh
python3.13 -m pip install wheel
pip install homeassistant==2025.3.1
```

6. Run the Home Assistant app with `/srv/homeassistant/bin/hass` and access it via browser in `<pi_ip>:8123`. Follow the steps to create an account.

## Setting up Home Assistant startup on boot

To make sure the home assistant app starts on boot, a service should be configured. Create the file `/etc/systemd/system/homeassistant.service` using `vim` or `nano` and copy the following contents inside:
```sh
[Unit]
Description=Home Assistant Service
PartOf=Tmux.service
After=Tmux.service

[Service]
User=homeassistant
Restart=no
RemainAfterExit=yes
WorkingDirectory=/srv/homeassistant
ExecStart=/usr/bin/tmux new-session -d -s homeassistant "/srv/homeassistant/bin/hass"

[Install]
WantedBy=multi-user.target
```
In order to enable the service (make sure no other instance of `hass` is running), run:
```sh
sudo systemctl daemon-reload
sudo systemctl start homeassistant
```
During development, it could be important to monitor log messages, which can be checked by accessing the `tmux` where home assistant is running:
```sh
tmux a -t homeassistant
```
To leave the current `tmux` session, type `Ctrl+B D`.

# VPN

(TBD)
- install wireguard
- set configurations (server and clients)
- set sofware for dynamically update ddns hostname (DUC from NO-IP, for example)
- set service for starting the wireguard interface automatically