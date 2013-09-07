IllinoisNet
===========

Get IllinoisNet working on linux!!  
First Copy the included wpa_supplicant to /etc/wpa_supplicant.conf  

You need to fill in your password and netid
To get a password do this
    password=hash:blah(you can generate your hash with ' echo -n password_here | iconv -t utf16le | openssl md4')
You need to fill in passoword feild in the config, this can be plain text or it can be a hask generated by the command above.

wpa_passphrase NetworkName NetworkPassword  
You can also add other routes by using the command above, the password will be encrypted  
It will generate something like this:  
    network={
    	ssid="OtherHomeNetwork"
    	psk=some_long+string you can generate with wpa_passphrase ssid password
    }

add more networks as needed in the wpa_supplicant.conf file  

To connect to the internet, put this in your ~/.bashrc or .zshrc and   
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

You might want to make this script run on boot.
