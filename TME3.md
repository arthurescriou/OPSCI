# TME 3 : Docker

## Containerize an application 

#### Get the app 

1. Clone the redis-node-app repository using the following command:

```bash
git clone https://github.com/arthurescriou/redis-node.git 
```

#### Build the app's image

2. In the terminal, run the following commands :
  
``` bash
cd /path/to/redis-node-app
```

3. Create an empty file named Dockerfile.

```` bash
touch Dockerfile
````
4. Using a text editor or code editor, add the following contents to the Dockerfile:

```` docker
FROM node:alpine
COPY . .
RUN yarn
EXPOSE 8080
CMD node main.js
````

5. Build the image. 
``` bash
docker build -t redis-node .
```

### Start an app container

6. Run your container using the docker run command and specify the name of the image you just created:
```
docker run -dp 127.0.0.1:3000:3000 redis-node
```

7. Run the following docker ps command in a terminal to list your containers.

``` bash
docker ps
```

Output similar to the following should appear :
```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
df784548666d        redis-node     "docker-entrypoint.sâ¦"   2 minutes ago       Up 2 minutes        127.0.0.1:3000->3000/tcp   priceless_mcclintock
```

### Update the application

#### Update the source code

8. In the src/static/js/app.js file, update line to use the new empty text.

9. Build your updated version of the image, using the docker build command.

``` docker
docker build -t redis-node  .
```

10. Start a new container using the updated code.
``` docker 
docker run -dp 127.0.0.1:3000:3000 redis-node 
```

You probably saw an error like this:
```
docker: Error response from daemon: driver failed programming external connectivity on endpoint laughing_burnell 
(bb242b2ca4d67eba76e79474fb36bb5125708ebdabd7f45c8eaf16caaabde9dd): Bind for 127.0.0.1:3000 failed: port is already allocated.
```

#### Remove the old container

11. Get the ID of the container by using the docker ps command.
```
docker ps
```

12. Use the docker stop command to stop the container. Replace <the-container-id> with the ID from docker ps
```
docker stop <the-container-id>
```

13. Once the container has stopped, you can remove it by using the docker rm command.
```
docker rm <the-container-id>
```

14. Now, start your updated app using the docker run command.
```
docker run -dp 127.0.0.1:3000:3000 redis-node 
```

## Multi container apps

### Redis

15. Create the network.
```
docker network create todo-app
```

16. Pull the redis Docker image.
```
docker pull redis
```

17. Start a redis container
```
docker run --name node-redis --network todo-app -d redis
```

### React

18. Clone the redis-react app 
```
git clone https://github.com/arthurescriou/redis-react.git
```
19. Create the redis-react dockerfile
```
FROM node:alpine
COPY . .
WORKDIR /path/to/redis-node-app 
RUN yarn
EXPOSE 3000
CMD CMD ["npm", "start"]
```

20. Build the image 

```
docker image build -t redis-react:latest .
```
21. Start a redis container
```
docker run --name redis-react --network todo-app -d redis
```

## Bonus Exercise 

Create a Dockerfile to launch the entire application using Docker Compose.