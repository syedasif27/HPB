# Installation and setup of selfhosted High performance Bakcend 
  
# 1. Install System Dependencies

sudo dnf install -y git gcc make curl \
  libevent libevent-devel openssl openssl-devel \
  libnice libnice-devel glib2 glib2-devel \
  libuuid-devel libtool automake autoconf

 # 2. Install Go Language

Check the latest version on https://go.dev/dl/

cd /tmp

curl -LO https://go.dev/dl/go1.22.2.linux-amd64.tar.gz

sudo rm -rf /usr/local/go

sudo tar -C /usr/local -xzf go1.22.2.linux-amd64.tar.gz

Add Go to your environment:

echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc

source ~/.bashrc

Verify:

go version

# 3. Clone the HPB Repository

cd /opt

sudo git clone https://github.com/nextcloud/spreed-signaling.git

cd spreed-signaling

# 4. Build the Signaling Server

make

It will create a binary file named signaling.

# 5. Create and Edit Config File

Copy example config:

cp signaling.conf.in signaling.conf

nano signaling.conf

Modify these key settings:

[http]
listen = :8080
tls = true
cert = /etc/ssl/certs/your_cert.crt
key = /etc/ssl/private/your_key.key

[app]
sharedsecret = your-signaling-secret

[turn]
servers = stun:your-turn-server:3478
secret = your-turn-secret

Replace:

    your_cert.crt / your_key.key with your SSL paths.

    your-signaling-secret and your-turn-secret with strong random strings.

# 6. Run the Signaling Server (Test Mode)

./signaling -config signaling.conf

Check that it starts without errors. You should see logs saying it's listening on port 8080 (or your chosen port).

# 7. (Optional) Setup as a systemd Service

Create the service file:

 sudo nano /etc/systemd/system/nextcloud-signaling.service

Paste the following:

[Unit]
Description=Nextcloud Talk Signaling Server
After=network.target

[Service]
ExecStart=/opt/spreed-signaling/signaling -config /opt/spreed-signaling/signaling.conf
WorkingDirectory=/opt/spreed-signaling/
Restart=always
User=nobody
Group=nogroup

[Install]
WantedBy=multi-user.target

Enable and start it:

sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable --now nextcloud-signaling

# 8. Allow Firewall Port (If Needed)

 sudo firewall-cmd --add-port=8080/tcp --permanent
 
 sudo firewall-cmd --reload

Or if you're using a reverse proxy (e.g., Nginx), allow 443 only.
# 9. Update Nextcloud Config

On your web server, edit:

 sudo -u apache php /var/www/html/nextcloud/occ config:system:set spreed:signaling_servers 0 --value="https://your-hpb-server"
 
 sudo -u apache php /var/www/html/nextcloud/occ config:system:set spreed:signaling_secret --value="your-signaling-secret"

# 10. Verify HPB is Working

Run:

 sudo -u apache php /var/www/html/nextcloud/occ talk:signaling:check

If it’s successful, you're all set!


# Diffeerence between Selfhosted HPB and Enterprise HPB
![Screenshot from 2025-04-21 15-53-00](https://github.com/user-attachments/assets/df352a60-94a0-4d89-a791-4809014ef25a)

# Difference between TURN and STUN server
![Screenshot from 2025-04-21 15-48-56](https://github.com/user-attachments/assets/471a1770-f96e-4387-86cb-71b94ab23853)

# Diagram of Mesh topology and SFU (Selective Forwarding Unit)
![ChatGPT Image Apr 21, 2025, 03_49_06 PM](https://github.com/user-attachments/assets/eca820c2-dea6-443d-af93-392e3e28e3a1)

# Difference between Mesh and SFU (Group call of 5 users)
![Screenshot from 2025-04-21 16-09-43](https://github.com/user-attachments/assets/3c6a6d86-0afc-49b5-9619-98e5bb6dc0c5)


