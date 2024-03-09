server {

if ($host = harv.sreesanth.me) { 
  return 301 https://$host$request_uri; 
}   # managed by Certbot

listen 80;
listen [::]:80;
server_name *.sreesanth.me; 
return 404; # managed by Certbot