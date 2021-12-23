## How to Get Let's Encrypt SSL on Ubuntu 20.04
https://serverspace.io/support/help/how-to-get-lets-encrypt-ssl-on-ubuntu/

#### Install letsencrypt
```
sudo apt install letsencrypt
```

#### Status of letsencrypt
```
sudo systemctl status certbot.timer
```

####  letsencrypt SSL certificate for Standalone server (No Web Server)
```
sudo certbot certonly --manual --agree-tos --preferred-challenges dns -d msrlabs.decode-iot.com
```

#### Move into certificate folder
```
/etc/letsencrypt/live/foldername
```

#### Convert pem files to pfx for Azure Application Gateway listeners 
```
openssl pkcs12 -export -out decode-ag-msrlabs.pfx -inkey privkey.pem -in cert.pem -certfile chain.pem
```

#### Note: Password
Password@123
