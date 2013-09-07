IllinoisNet
===========

#Get IllinoisNet working on linux!!  
**You need to fill in your password and netid in the config file  
First Copy the included wpa_supplicant to /etc/wpa_supplicant.conf**   

To get a password do `echo -n password_here | iconv -t utf16le | openssl md4`
    password=hash:MY_HASH_HERE

Your password could also be in plaintext. You should be careful with the permissions of the file. They should be restricted so random people dont read your file over the internet!

##To add other networks

`wpa_passphrase NetworkName NetworkPassword`

You can also add other routes by using the command above, the password will be encrypted  
It will generate something like this:  
```bash
    network={
    	ssid="OtherHomeNetwork"
    	psk=some_long+string you can generate with wpa_passphrase ssid password
    }
```
add more networks as needed in the wpa_supplicant.conf file  

To connect to the internet, put this in your ~/.bashrc or .zshrc and   
```bash
    function internet(){
      sudo service network-manger stop
      sudo service wicd stop
      sudo killall wpa_supplicant
      sudo wpa_supplicant -i wlan0 -c /etc/wpa_supplicant.conf -B # -b for background you may need to change this to wlan1 based on what sudo iwconfig says
      dhclient -r wlan0 # release old connection, again may want to change to wlan1
      echo "refreshing the dhcp client - this could take a while or it could not work"
      sudo dhclient wlan0
      sudo iwconfig
      echo "and we are good to go - happy surfing"
    }
    function refresh(){
      sudo dhclient wlan0
    }
```
You might want to make this script run on boot.
