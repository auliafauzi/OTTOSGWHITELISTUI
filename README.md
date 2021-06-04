# OttoSG_WebPortal

Web Portal For OttoSG Whitelist

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080 for development sake
npm run serve

# build for production
npm run build
# The HTML assets are built under dist folder
# for nginx server, copy all items inside ./dist folder to /var/www/html/{your_project_name}

#Copy these into your nginx conf under /etc/nginx/sites-enabled/
location /{your_project_name} {
          root "/var/www/html";
          try_files $uri $uri/ /index.html;
        }


```
``` bash
API url is defined at ./src/utils.js file, on BASE_URL variable
```
