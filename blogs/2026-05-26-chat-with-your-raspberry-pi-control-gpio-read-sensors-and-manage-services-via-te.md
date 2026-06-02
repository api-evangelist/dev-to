---
title: Chat With Your Raspberry Pi — Control GPIO, Read Sensors, and Manage Services
  via Telegram Using Garudust
url: https://forem.com/garudust/chat-with-your-raspberry-pi-control-gpio-read-sensors-and-manage-services-via-telegram-using-40d7
date: '2026-05-26'
author: Garudust
feed_url: https://forem.com/feed
---
Your Raspberry Pi is sitting at home doing something useful — logging sensor data, driving a relay, running a home assistant script. You're out. You want to check on it. You used to SSH in from your laptop. Today you just open Telegram and ask. You:    is the temperature sensor still working?
Agent:  ✓ ran: cat /sys/bus/w1/devices/28-*/w1_slave
        Current temperature: 24.3°C — sensor is responding normally.

You:    turn the fan relay on
Agent:  ⚠ This will trigger GPIO 18 — confirm? [y/n]
You:    y
Agent:  ✓ ran: python3 ~/scripts/relay.py on 18
        Relay ON. Fan is running. This is Garudust Agent with an SSH sandbox. The agent lives on a cloud VPS (or any server with internet access). You chat via Telegram. The agent SSHes into your Pi and runs whatever it needs to. Your Pi only needs SSH enabled — no extra software, no port forwarding for the agent. How It Works You (Telegram)
    │
    ▼
garudust-server          ← runs on a VPS or home server
(Telegram adapter)
    │
    │  SSH  (key auth, BatchMode, StrictHostKeyChecking)
    ▼
Raspberry Pi             ← just needs sshd running
(GPIO, sensors, scripts) The agent's terminal tool normally runs commands locally. With terminal_sandbox: ssh , every command is forwarded through ssh to the Pi instead. The agent never knows or cares — it still calls the same tool. The routing happens transparently in the sandbox layer. Prerequisites A Raspberry Pi with SSH enabled (any model, any recent Raspberry Pi OS) A server to run Garudust on — a $5 VPS, another machine on your network, or even the Pi itself pointing at localhost A Telegram bot token (from @BotFather ) Garudust installed on the server # On the server — install Garudust curl -LO "https://github.com/garudust-org/garudust-agent/releases/latest/download/garudust- $( curl -s https://api.github.com/repos/garudust-org/garudust-agent/releases/latest | grep tag_name | cut -d '"' -f4 ) -x86_64-unknown-linux-musl.tar.gz" tar -xzf garudust- * .tar.gz && sudo mv garudust garudust-server /usr/local/bin/ Step 1 — Set Up a Dedicated SSH Key Never use your personal SSH key for an agent. Create a dedicated one on the server: ssh-keygen -t ed25519 -f ~/.ssh/garudust_pi -N "" -C "garudust-agent" Copy the public key to the Pi: ssh-copy-id -i ~/.ssh/garudust_pi.pub pi@192.168.1.50 # or if Pi is not on the same network yet: # cat ~/.ssh/garudust_pi.pub | ssh pi@<pi-ip> "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys" Verify it works without a password prompt: ssh -i ~/.ssh/garudust_pi -o BatchMode = yes pi@192.168.1.50 "echo OK" # → OK Step 2 — Configure Garudust ~/.garudust/.env : ANTHROPIC_API_KEY = sk-ant-... # or any supported provider TELEGRAM_TOKEN = 123456789:AAFxxx # your bot token ~/.garudust/config.yaml : security : approval_mode : smart # read-only commands auto-approve, writes ask first terminal_sandbox : ssh ssh_host : " 192.168.1.50" # your Pi's IP (or hostname) ssh_user : " pi" ssh_port : 22 ssh_key_path : /home/youruser/.ssh/garudust_pi platforms : telegram : enabled : true port : 8443 webhook_path : /telegram roles : default_role : ~ # strangers get no access definitions : owner : approval_mode : auto # full auto-approve for you users : telegram : " YOUR_TELEGRAM_USER_ID" : owner Find your Telegram user ID by messaging @userinfobot . Start the server: garudust-server What You Can Now Do Read Sensor Data DS18B20 temperature sensor (1-Wire): You:    temperature?
Agent:  24.3°C You:    log temperature every 10 seconds for 1 minute and tell me the average
Agent:  ✓ ran: for i in $(seq 6); do cat /sys/bus/w1/devices/28-*/w1_slave | grep t= | awk -F= '{print $2/1000}'; sleep 10; done
        Readings: 24.1, 24.2, 24.3, 24.3, 24.4, 24.2
        Average: 24.25°C DHT22 humidity + temperature: You:    humidity and temp from the DHT22
Agent:  ✓ ran: python3 ~/scripts/dht22_read.py
        Temperature: 24.3°C  Humidity: 58.2% Control GPIO (Relay, LED, Motor) You:    turn on the garden lights (relay on GPIO 18)
Agent:  ⚠ This will modify GPIO state — confirm?
You:    yes
Agent:  ✓ ran: python3 ~/scripts/relay.py on 18
        GPIO 18 HIGH — relay activated.

You:    set the LED strip to blue
Agent:  ✓ ran: python3 ~/scripts/led_strip.py color 0 0 255
        Done.

You:    turn everything off
Agent:  ✓ ran: python3 ~/scripts/all_off.py
        All GPIO outputs set LOW. Check System Health You:    how is the Pi doing?
Agent:  ✓ ran: vcgencmd measure_temp && free -h && df -h / && uptime
        CPU temp:    52.1°C
        RAM:         823 MB used / 1.9 GB total
        Disk:        12 GB used / 29 GB total (41%)
        Uptime:      14 days, 3 hours

You:    is anything hogging the CPU?
Agent:  ✓ ran: ps aux --sort=-%cpu | head -6
        python3 sensor_logger.py   4.2%
        node    homebridge          2.8%
        Everything else < 1%. Manage Services You:    restart homebridge
Agent:  ⚠ This will restart the homebridge service — confirm?
You:    y
Agent:  ✓ ran: sudo systemctl restart homebridge
        homebridge restarted. Status: active (running).

You:    is the camera stream still up?
Agent:  ✓ ran: systemctl is-active mjpg-streamer
        active

You:    show me the last 20 lines of the sensor log
Agent:  ✓ ran: tail -20 ~/logs/sensor.log
        [2026-05-26 14:01] temp=24.3 hum=58.2
        [2026-05-26 14:02] temp=24.4 hum=57.9
        ... Run Your Own Scripts The agent can run any script already on the Pi. You don't need to teach it the commands — just describe what you want: You:    water the plants for 30 seconds
Agent:  ✓ ran: python3 ~/scripts/sprinkler.py on 30
        Valve opened. Will close in 30 seconds.

You:    take a photo and tell me if the bird feeder is empty
Agent:  ✓ ran: raspistill -o /tmp/feeder.jpg -w 1280 -h 720
        [reads image]
        The feeder appears about 30% full — seeds visible on the left side. Automate With Cron Set recurring checks without staying in the chat: # config.yaml cron : timezone : " Asia/Bangkok" jobs : - schedule : " 0 8 * * *" task : " check Pi health and send a morning summary to Telegram" - schedule : " */30 * * * *" task : " read the temperature sensor; if above 70°C send a warning to Telegram" Now the agent proactively messages you — no need to ask. Security Notes A few things worth knowing before you deploy this: The SSH connection is hardened automatically. Garudust uses BatchMode=yes (no password prompts), StrictHostKeyChecking=accept-new (rejects changed host keys to prevent MITM), and clears the subprocess environment before spawning ssh so no API keys or secrets reach the Pi. Approval mode gates destructive commands. With approval_mode: smart , read-only commands (reading sensors, checking logs, git status ) run without asking. Anything that modifies state asks for confirmation first. Set approval_mode: auto only for yourself after you've tested it. Lock down the pi user if you want to go further. Restrict the SSH key to specific commands using command= in authorized_keys , or run the agent's SSH user in a dedicated account with only the scripts it needs to call. The Pi never exposes a port to the internet. The agent SSHes out from the server to the Pi — the Pi only needs port 22 open to your server, not to the world. What's Next Multiple Pis — point different agent sessions at different hosts; or build a thin wrapper script that SSHes to the right Pi based on what you ask Voice control — pair with iApp TTS to get spoken responses back on a speaker connected to the Pi Skill learning — after you run a workflow a few times, Garudust automatically writes it as a reusable skill so you don't need to describe it step by step anymore The full source for Garudust is at garudust-org/garudust-agent . The SSH sandbox landed in v0.13 — grab the latest binary from Releases or build from source with cargo build --release .
