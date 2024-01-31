# TME 3 : Docker

## Requirement 

1. Create the network.


```
docker network create redis-app
```

{:start="2"}
2. Pull the redis Docker image. Go the DockerHub website https://hub.docker.com/_/redis


```
docker pull redis
```
  
{:start="3"}
3. Start a redis container


```
docker run --name node-redis --network todo-app -d redis
```


## Containerize an application 

#### Get the app 

{:start="4"}
4. Clone the redis-node-app repository using the following command:

```bash
git clone https://github.com/arthurescriou/redis-node.git 
```

#### Build the app's image

{:start="5"}
5. In the terminal, run the following commands :
  
``` bash
cd /path/to/redis-node-app
```

{:start="6"}
6. Create an empty file named Dockerfile.

```` bash
touch Dockerfile
````

{:start="7"}
7. Using a text editor or code editor, add the following contents to the Dockerfile:


```` docker
FROM node:18-alpine
COPY . .
RUN yarn
EXPOSE 8080
CMD node main.js
````

{:start="8"}
8. Build the image. 


``` bash
docker build -t redis-node .
```

**Note** :
The docker build command uses the Dockerfile to build a new image. You might have noticed that Docker downloaded a lot of "layers". This is because you instructed the builder that you wanted to start from the node:18-alpine image. But, since you didn't have that on your machine, Docker needed to download the image.

After Docker downloaded the image, the instructions from the Dockerfile copied in your application and used yarn to install your application's dependencies. The CMD directive specifies the default command to run when starting a container from this image.

Finally, the -t flag tags your image. Think of this as a human-readable name for the final image. Since you named the image getting-started, you can refer to that image when you run a container.

The . at the end of the docker build command tells Docker that it should look for the Dockerfile in the current directory

### Start an app container

{:start="9"}
9. Run your container using the docker run command and specify the name of the image you just created:


```
docker run -dp 127.0.0.1:3000:3000 redis-node
```

**Note** :
The -d flag (short for --detach) runs the container in the background. This means that Docker starts your container and returns you to the terminal prompt. You can verify that a container is running by viewing it in Docker Dashboard under Containers, or by running docker ps in the terminal.

The -p flag (short for --publish) creates a port mapping between the host and the container. The -p flag takes a string value in the format of HOST:CONTAINER, where HOST is the address on the host, and CONTAINER is the port on the container. The command publishes the container's port 3000 to 127.0.0.1:3000 (localhost:3000) on the host. Without the port mapping, you wouldn't be able to access the application from the host.


{:start="10"}
10. Run the following docker ps command in a terminal to list your containers.


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

{:start="11"}
11. In the src/static/js/app.js file, update line to use the new empty text.


{:start="12"}
12. Build your updated version of the image, using the docker build command.

``` docker
docker build -t redis-node  .
```

{:start="13"}
13. Start a new container using the updated code.


``` docker 
docker run -dp 127.0.0.1:3000:3000 redis-node 
```

You probably saw an error like this:
```
docker: Error response from daemon: driver failed programming external connectivity on endpoint laughing_burnell 
(bb242b2ca4d67eba76e79474fb36bb5125708ebdabd7f45c8eaf16caaabde9dd): Bind for 127.0.0.1:3000 failed: port is already allocated.
```

#### Remove the old container

{:start="14"}
14. Get the ID of the container by using the docker ps command.


```
docker ps
```

{:start="15"}
15. Use the docker stop command to stop the container. Replace <the-container-id> with the ID from docker ps


```
docker stop <the-container-id>
```

{:start="16"}
16. Once the container has stopped, you can remove it by using the docker rm command.


```
docker rm <the-container-id>
```

{:start="17"}
17. Now, start your updated app using the docker run command.


```
docker run -dp 127.0.0.1:3000:3000 redis-node 
```

## Multi container apps

### React

{:start="18"}
18. Clone the redis-react app 


```
git clone https://github.com/arthurescriou/redis-react.git
```

{:start="19"}
19. Create the redis-react dockerfile


```
A FAIRE
```

{:start="20"} 
20. Build the image 


```
docker image build -t redis-react:latest .
```

{:start="21"}
21. Start a redis container


```
docker run --name redis-react --network redis-app -d redis
```

## Bonus Exercise 

Create a Dockerfile to launch the entire application using Docker Compose.
