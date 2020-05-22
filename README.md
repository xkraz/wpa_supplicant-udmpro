# wpa_supplicant for UDM Pro
## For 1.7

ALL CREDIT TO PBRAH, GiulianoM, & fryjr82 

```
## overview
This guide has primarily been written for authenticating to AT&T U-Verse using wpa_supplicant on a UDM Pro.  It is assumed that you've already retrieved your certificates from a modem supplied by AT&T.  If you have not, you can purchase a used modem on ebay, such as the NVG589 and then root it to get the certificates.  I had success using the following guide.


https://github.com/bypassrg/att


If the above link no longer works, I have also forked it to my GitHub below.

https://github.com/pbrah/att



## running the docker image
1.  scp your certs and wpa_supplicant.conf to the UDM Pro

```
#scp -r *.pem root@192.168.1.1:/tmp/

root@192.168.1.1's password:
CA_001E46-xxxx.pem                                                          100% 3926     3.8KB/s   00:00
Client_001E46-xxxx.pem                                                      100% 1119     1.1KB/s   00:00
PrivateKey_PKCS1_001E46-xxxx.pem                                            100%  887     0.9KB/s   00:00

#scp -r wpa_supplicant.conf root@192.168.1.1:/tmp/

wpa_supplicant.conf                                                         100%  680     0.7KB/s   00:00
```

2. ssh to the UDM Pro, create a directory for the certs and wpa_supplicant.conf in the podman directory then copy the files over.

```
#mkdir /mnt/data/podman/wpa_supplicant/

#cp -arfv /tmp/*pem /tmp/wpa_supplicant.conf /mnt/data/podman/wpa_supplicant/

```

3. Update the wpa_supplicant.conf to reflect the correct paths for our container.  **Do not run these more than once or you will end up with incorrect paths.**

```
#sed -i 's,ca_cert=",ca_cert="/etc/wpa_supplicant/conf/,g' /mnt/data/podman/wpa_supplicant/wpa_supplicant.conf
#sed -i 's,client_cert=",client_cert="/etc/wpa_supplicant/conf/,g' /mnt/data/podman/wpa_supplicant/wpa_supplicant.conf
#sed -i 's,private_key=",private_key="/etc/wpa_supplicant/conf/,g' /mnt/data/podman/wpa_supplicant/wpa_supplicant.conf
```

After running the sed commands, verify your paths in wpa_supplicant.conf look something like this:
```
# cd /mnt/data/podman/wpa_supplicant
# cat wpa_supplicant.conf
# Generated by 802.1x Credential Extraction Tool
# Copyright (c) 2018-2019 devicelocksmith.com
# Version: 1.04 linux amd64
#
# Change file names to absolute paths
eapol_version=1
ap_scan=0
fast_reauth=1
network={
        ca_cert="/etc/wpa_supplicant/conf/CA_001E46-xxxxxxxx.pem"
        client_cert="/etc/wpa_supplicant/conf/Client_001E46-xxxxxx.pem"
        eap=TLS
        eapol_flags=0
        identity="10:05:B1:xx:xx:xx" # Internet (ONT) interface MAC address must match this value
        key_mgmt=IEEE8021X
        phase1="allow_canned_success=1"
        private_key="/etc/wpa_supplicant/conf/PrivateKey_PKCS1_001E46-xxxxxx.pem"
}
# WARNING! Missing AAA server root CA! Add AAA server root CA to CA_001E46-xxxxxx.pem
#
```

4. Run the wpa_supplicant podman container, the podmanr run command below assumes you are using port 9 WAN.  If not, adjust accordingly.

```
#podman run --privileged=true --network=host --name=wpa_supplicant-udmpro -v /mnt/data/podman/wpa_supplicant/:/etc/wpa_supplicant/conf/ --log-driver=k8s-file --restart=on-failure -detach -ti pbrah/wpa_supplicant-udmpro:v1.0 -Dwired -ieth8 -c/etc/wpa_supplicant/conf/wpa_supplicant.conf
```

## troubleshooting
If you are having issues connecting after starting your docker container, the first thing you should do is check your docker container logs.
```
docker logs -f wpa_supplicant-udmpro
```
