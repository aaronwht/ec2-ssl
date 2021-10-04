# EC2 SSL

## Install nvm and certbot
`sudo su`  
`amazon-linux-extras install nginx1.12 epel -y`  
`yum install certbot -y`    
`yum update -y`  

## Create SSL Certificate
`certbot certonly --standalone -d YOUR_WEBSITE --agree-to --debug --email email@address.com --no-eff-email`  

## Install Node.js
`curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash`  

### Must exit EC2 instance and re-SSH into it  

`nvm i node`  
`npm i -g pm2` 

## Create a Node.js File  

`sudo nano app.js` 
```
const http = require('http');

const headers = {
    'Access-Control-Allow-Origin': '*',
    'Access-Control-Allow-Methods': 'OPTIONS, POST, GET',
    'Access-Control-Max-Age': 2592000, // 30 days
};

http.createServer((req, res) => {
    res.setHeader('Access-Control-Allow-Origin', '*');
    if (req.method === 'OPTIONS') {
        res.writeHead(204, headers);
        res.end();
        return;
    }

    if (['GET', 'POST', 'PUT'].indexOf(req.method) > -1) {
        req.on('data', chunk => {
            console.log(chunk.toString());
        });

        res.writeHead(200, headers);
        res.end('success');
        return;
    }

    res.writeHead(405, headers);
    res.end(`${req.method} is not allowed for the request.`);
}).listen(8000);
```  

`pm2 start app.js` 

## Configure NGINX to serve site via SSL  
`sudo nano /etc/nginx/nginx.conf`  

Update server name to your URL and redirect non-SSL traffic to SSL.
```
server {
   listen 80
   server_name YOUR_WEBSITE;
   location / {
      return 301 https://$server_name$request_uri;
   }
}
```  

## Add a handler for the SSL traffic:
```
server {
    listen 443 ssl;
    server_name YOUR_WEBSITE;
    ssl_certificate /etc/letsencrypt/live/YOUR_WEBSITE/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/YOUR_WEBSITE/privkey.pem;
    add_header Strict-Transport-Security “max-age=31536000”;
    location / {
      proxy_pass http://127.0.0.1:8000;
    }
}

```

`sudo service nginx start`
