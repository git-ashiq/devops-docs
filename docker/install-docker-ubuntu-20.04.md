## How To Install and Use Docker on Ubuntu 20.04
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04

#### Create the docker group if it does not exist
```
sudo groupadd docker
```

#### Add your user to the docker group
```
sudo usermod -aG docker $USER
```

#### Run the following command or Logout and login again and run
```
newgrp docker
```

#### Check if docker can be run without root
```
docker run hello-world
```

#### Reboot if still got error
```
reboot
```
