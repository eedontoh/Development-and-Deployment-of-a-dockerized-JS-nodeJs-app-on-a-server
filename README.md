## Developing and deploying a Dockerized JS app on a server

Project Description

This demo app shows a simple user profile app set up development and deployment using 
- index.html with pure js and css styles
- nodejs backend with express module
- mongodb for data storage
- Docker for containerization
- Amazon ECR as private repo
- MongoExpress as DB UI



All components are docker-based

### With Docker

#### To start the application

Step 1: Create docker network

    docker network create mongo-network 

Step 2: start mongodb 

    docker run -d -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=password --name mongodb --net mongo-network mongo    

Step 3: start mongo-express
    
    docker run -d -p 8081:8081 -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin -e ME_CONFIG_MONGODB_ADMINPASSWORD=password --net mongo-network --name mongo-express -e ME_CONFIG_MONGODB_SERVER=mongodb -e ME_CONFIG_MONGODB_URL=mongodb://mongodb:27017 mongo-express   

_NOTE: creating docker-network in optional. You can start both containers in a default network. In this case, just emit `--net` flag in `docker run` command_

Step 4: open mongo-express from browser

    http://localhost:8081

Step 5: create `user-account` _db_ and `users` _collection_ in mongo-express

<img width="1680" height="1050" alt="Screenshot 2026-09-05 at 13 23 54" src="https://github.com/user-attachments/assets/e5550e7b-9f02-49e6-8b77-df8a09319002" />


Step 6: Start your nodejs application locally - go to `app` directory of project 

    cd app
    npm install 
    node server.js

<img width="434" height="60" alt="started node app" src="https://github.com/user-attachments/assets/4ce6b929-db58-4424-a26f-4763e9cfa8ce" />


Step 7: Access you nodejs application UI from browser

    http://localhost:3000


<img width="1680" height="1050" alt="running app" src="https://github.com/user-attachments/assets/1d6d6e7d-5e4d-4cbe-9a75-fb587631f970" />

### With Docker Compose

#### To start the application

Step 1: start mongodb and mongo-express

    docker-compose -f docker-compose.yaml up
    
_You can access the mongo-express under localhost:8080 from your browser_
    
Step 2: in mongo-express UI - create a new database "user-account"

Step 3: in mongo-express UI - create a new collection "users" in the database "user-account"       
    
Step 4: start node server 

    cd app
    npm install
    node server.js
    
Step 5: access the nodejs application from browser 

    http://localhost:3000

#### To build a docker image from the application

    docker build -t my-app:1.0 .       
    
The dot "." at the end of the command denotes location of the Dockerfile.


step 7: Deploy Docker application on a server with Docker Compose


i. Copy Docker-compose file to remote server

ii. Login to private Docker registry on remote server to fetch our app image

iii. Start our application container with MongoDB and MongoExpress services using docker compose


Step 1:

Create an image of the application in the app folder and push it to the private registry:
Because the URL to access the mongo-db from the application is different when running the application in the same Docker network as the mongo-db (use mongodb://admin:password@mongodb instead of mongodb://admin:password@localhost:27017 or mongodb://admin:password@host.docker.internal:27017) we have to build a new version of the image (containing the adjusted server.js file) and push it to the private registry.

    cd app
    

Step 2: login to private registry
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 653455484040.dkr.ecr.us-east-1.amazonaws.com



Step 3:

a. build the image

    docker build -t my-app:1.0 .

b. tag the image
docker tag my-app:1.0 653455484040.dkr.ecr.us-east-1.amazonaws.com/my-app:1.1

c. push it to the registry
docker push 653455484040.dkr.ecr.us-east-1.amazonaws.com/my-app:1.1

<img width="1680" height="1050" alt="image versions in ecr" src="https://github.com/user-attachments/assets/2aa4a2d5-5375-4fa6-baf0-73e6cc2ab3df" />


Step 4: Copy the docker-compose.yaml file:

Switch to a server / directory where you want to run the application and copy the docker-compose.yaml file into this directory.

Step 6: Run the application
a. login to a private registry
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 653455484040.dkr.ecr.us-east-1.amazonaws.com

b. run docker-compose
docker-compose up -d


Test the application in the browser (http://localhost:3000, http://localhost:8081).


<img width="1680" height="1050" alt="running app" src="https://github.com/user-attachments/assets/cef4daf7-bff6-4201-a267-6d350cd1d3c8" />
