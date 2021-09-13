# Deploy angular for dev and prod environments

Github: git@github.com:lander22/DevOpsCert.git

We are gonna do:

- Create Dockerfile for dev environment: using Angular CLI + node.js
- Create Dockerfile for pro environment: using Angular CLI + node.js + nginx + lighter image than the dev one
- Create Docker-compose to run both services and create images: add a volume for dev porpouse and expose both services at local host to check them.





# Create DockerfileDev

As Angular CLI has its own develop environment, with "ng serve", let's create an image and host it at local:

FROM node:16.9.0-slim 

RUN mkdir -p /app

WORKDIR /app

COPY package.json /app

RUN npm install

COPY . /app

RUN npm install @angular/cli -g

CMD ["ng","serve","--host","0.0.0.0"]


# Create Docker-compose for dev 

Now let's create docker-compose.yml exposing ports and adding volumes to develop changes in a easier way. I just add my source code path as the volume, YOU HAVE TO SET YOUR OWN PATH. ( in the code I comment my own path to be used for everyone ) 

version: "3.9"

services:
  angularprojectdev:
    image: lander22/angularprojectsimplilearn:dev
    build:
      context: ..
      dockerfile: deployment/DockerfileDev
    ports:
      - "4201:4200"
    volumes:
      - /home/eellaandeergmai/angularProject/first-project/src:/app/src
      

run "docker-compose -f deployment/docker-compose build" 

"docker push lander22/angularprojectsimplilearn:dev"


The image is successfully created. But a bit heavy. 

Now just run "docker-compose -f deployment/docker-compose up -d" and container dev starts at localhost:4201







# Create Dockerfile for Pro

As Angular CLI has "ng build" to create a development files, we can use it to create more lighter image and ready to be used. Let's take advantage of this and create the Dockerfile with 2 stages:


##Creating compiled code from angular

FROM node:16.9.0-slim AS base

RUN mkdir -p /app

WORKDIR /app

COPY package.json /app

RUN npm install

COPY . /app

RUN npm install @angular/cli -g

RUN ng build


##Add only dist folder to the nginx server

FROM nginx:1.21.3

WORKDIR /app

COPY --from=base /app/dist/first-project/ /var/www/html

COPY deployment/sites-availableConfig /etc/nginx/sites-available/default

COPY deployment/nginx.conf /etc/nginx/nginx.conf





We add some nginx configuration to run the server correctly and adding our sites-available location to the nginx.conf file. 



# Create Docker-compose for Pro

Now let's create our docker-compose for pro:

version: "3.9"

services:
    
  angularproject:
    image: lander22/angularprojectsimplilearn:pro
    build:
      context: ..
      dockerfile: deployment/Dockerfile
    ports:
      - "4202:4200"     
      
      
      
run "docker-compose -f deployment/docker-compose build" 

"docker push lander22/angularprojectsimplilearn:pro"

And the image is successfully created. But now much more lighter. 

Now just run "docker-compose -f deployment/docker-compose up -d" and container nginx with angular starts at localhost:4202.


# Annotations
      
I have create 2 docker compose but you can create one and up or build whatever service you need, as I did in the code linked.
In order to have a better dev environment, I set my own path, but you can take the container's path or yours path code. 
I just used nginx because I could create a lighter image taking advantage of ng build output.
