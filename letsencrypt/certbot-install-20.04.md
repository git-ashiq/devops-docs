# How to Get Let's Encrypt SSL on Ubuntu 20.04
https://serverspace.io/support/help/how-to-get-lets-encrypt-ssl-on-ubuntu/

# Install letsencrypt
sudo apt install letsencrypt

# Status
sudo systemctl status certbot.timer

# Standalone server for getting the "Let's Encrypt" SSL certificate
sudo certbot certonly --manual --agree-tos --preferred-challenges dns -d oxy.decode-iot.com

# Convert pem files to pfx for Azure Application Gateway listeners 
openssl pkcs12 -export -out decode-ag.pfx -inkey privkey.pem -in cert.pem -certfile chain.pem