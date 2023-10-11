# bose_qc
Downgrade your Bose QC35 from 3.0.3 to 2.5.5 or your Bose QC35 II from 4.5.2 to 4.3.6

I've made this video because I recently upgraded my Bose QC 35 to firmware 3.0.3 and noticed an issue with ANC not working properly anymore.

I checked on Internet how to downgrade, and finally found this Github project https://github.com/bosefirmware/ced 

==- Thank you guys for your amazing work! -==

I tried their methods to downgrade but none of them was working as Bose updated their Bose Updater software to a more recent version. 

I then found in the issues of this same project a topic dedicated to another option https://github.com/bosefirmware/ced/i...

I wasted two days before making it work, so I thought it would be interesting to share with you how to do it.


#####################################
#                   0. Credits                                              
#####################################

After several days trying to make the tutorial from https://github.com/bosefirmware/ced/i... work, I decided to make this video to help people.

Thank you guys for your work, and thank you lipov3cz3k for pointing out this new option to downgrade.

Thanks to pavel-d who added a comment helping me succeed in addition to the tutorial

#####################################
#                   1. Install NGINX                        
#####################################

(Not shown in the video)

brew install nginx

#####################################
#                   2. Test NGINX                            
#####################################

Start NGINX
brew services start nginx

Make sure NGINX is working by checking the url below
http://localhost:8080

Then stop NGINX
brew services stop nginx

#####################################
#                   3. Generate certificate                
#####################################

First, make a copy of the current SSL configuration file
cp /System/Library/OpenSSL/openssl.cnf ~/openssl-temp.cnf

Then edit the copy to add subjectAltName statement under v3_ca block
nano ~/openssl-temp.cnf

subjectAltName = DNS:*.bose.com

Now you're ready to generate the certificate
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt -config ~/openssl-temp.cnf

Leave every field with the default value, except Common Name, you must enter *.bose.com

#####################################
#                   4. Permission & Import              
#####################################

Change permission of generated self signed certificate
chmod 777 /etc/ssl/private/nginx-selfsigned.key
chmod 777 /etc/ssl/certs/nginx-selfsigned.crt 

Now add the self-signed certificate to Keychain in System certificates, and Always trust it

#####################################
#                   5. Modify your host file              
#####################################

Redirect all connections to worldwide.bose.com to localhost, so edit your host file
nano /etc/hosts

And then insert this line at the end of the file
127.0.0.1 worldwide.bose.com

#####################################
#                   6. Update NGINX conf              
#####################################

First make a copy of the original configuration file
cp /usr/local/etc/nginx/nginx.conf /usr/local/etc/nginx/nginx.conf.orig

Then edit the configuration file
nano /usr/local/etc/nginx/nginx.conf

And add this block to handle https connections
server { 
listen 443 default ssl; 
ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt; 
ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key; 

location = /connected_device { 
proxy_buffering off; 
proxy_pass https://raw.githubusercontent.com/bos... } 

location / { 
proxy_ssl_server_name on; 
proxy_buffering off; 
proxy_pass https://worldwide.bose.com/; 
}
}

#####################################
#                   7. Start NGINX                          
#####################################

You're almost done, you can start NGINX
brew services start nginx

You can check it's working by hitting https://worldwide.bose.com/connected_..., you should see the XML file from the github project https://raw.githubusercontent.com/bos...

If it's not working, check the error log to see if there's something wrong
tail -f /usr/local/var/log/nginx/error.log

You can also check logs to make sure Bose Updater is hitting your local NGINX
tail -f /usr/local/var/log/nginx/access.log

#####################################
#                   8. Downgrade                            
####################################

You can now go to https://btu.bose.com

When prompted launch the Bose Updater app

When your device is ready, press the following key combination: 'a' 'd' 'v' 'up arrow' 'down arrow' and then you'll be able to select the firmware you want.

Good luck!
