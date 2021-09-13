#Creating compiled code from angular

FROM node:16.9.0-slim AS base

RUN mkdir -p /app

WORKDIR /app

COPY package.json /app

RUN npm install

COPY . /app

RUN npm install @angular/cli -g

RUN ng build


#Add only dist folder to the nginx server

FROM nginx:1.21.3

WORKDIR /app

COPY --from=base /app/dist/first-project/ /var/www/html

COPY deployment/sites-availableConfig /etc/nginx/sites-available/default

COPY deployment/nginx.conf /etc/nginx/nginx.conf