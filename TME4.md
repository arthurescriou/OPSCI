# TME4 : Kubernetes

### Part 1: Application Containerization

#### Containerize a Simple Web Application:

1. Create a simple web application (e.g., a "Hello, World!" app using Node.js, Python Flask, or any other lightweight framework). cf Exersice 1
2. Write a Dockerfile to containerize this application.
3. Build the Docker image and push it to a container registry (e.g., Docker Hub, Google Container Registry).

**React-node dockerfile**

```sh
# Use the official Node.js 18 as a parent image
FROM node:18

# Set the working directory in the container
WORKDIR /usr/src/app

# Copy the current directory contents into the container at /usr/src/app
COPY . .

# Install any needed packages specified in package.json
RUN npm install

# Make port 3000 available to the world outside this container
EXPOSE 3000

# Run the app when the container launches
CMD ["npm", "start"]
```

Bonus : Launch app using docker-compose.yml

```yml
version: '3'
services:
  redis:
    image: redis
    networks:
      - redis

  redis-node:
    build:
      context: .
      dockerfile: Dockerfile
    environment:
      - REDIS_URL=redis://redis:6379
      - PORT=8080
    ports:
      - '8080:8080'
    depends_on:
      - redis
    networks:
      - redis

  redis-react:
    build:
      context: ../redis-react
      dockerfile: Dockerfile
    ports:
      - '3000:80'
    depends_on:
      - redis-node
networks:
  redis:
    driver: bridge
```

**Node.js 'Hello, World!' Application:**

- Create a file named app.js:

```javascript
const express = require('express')
const app = express()
const PORT = process.env.PORT || 3000

app.get('/', (req, res) => {
  res.send('Hello, World!')
})

app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`)
})
```

- Create a package.json file:

```json
{
  "name": "hello-world-app",
  "version": "1.0.0",
  "description": "A simple Node.js app",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.17.1"
  }
}
```

- Dockerfile:

```sh
# Use an official Node runtime as a parent image
FROM node:18

# Set the working directory in the container
WORKDIR /usr/src/app

# Copy the current directory contents into the container at /usr/src/app
COPY . .

# Install any needed packages specified in package.json
RUN npm install

# Make port 3000 available to the world outside this container
EXPOSE 3000

# Define environment variable
ENV NAME World

# Run app.js when the container launches
CMD ["node", "app.js"]
```

### Part 2: Deployment with Kubernetes

### Create a Deployment:

4. Write a Kubernetes deployment YAML file to deploy your containerized application. Specify the Docker image you pushed to the registry.

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-node
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-world-node
  template:
    metadata:
      labels:
        app: hello-world-node
    spec:
      containers:
        - name: hello-world-node
          image: yourusername/hello-world-node
          ports:
            - containerPort: 3000
```

6. Use kubectl apply -f your_deployment_file.yaml to create the deployment in your cluster.

### Expose Your Application:

6. Expose your application to the Internet by creating a Kubernetes service. You can start with a ClusterIP service and then upgrade it to a LoadBalancer or use an Ingress for more advanced scenarios.

```yml
apiVersion: v1
kind: Service
metadata:
  name: hello-world-node-service
spec:
  selector:
    app: hello-world-node
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer
```

8. Use kubectl expose deployment your_deployment_name --type=LoadBalancer --name=your_service_name to expose your deployment.

### Part 3: Scaling and Updating

#### Scaling the Application:

8. Scale your deployment to run multiple replicas of your application. Consider the load and how many instances you need to handle it.
9.
10. Use kubectl scale deployment your_deployment_name --replicas=3 to scale your deployment to 3 replicas.

#### Updating the Application:

10. Make a small change to your web application (for example change the message or add a new feature).

11. Rebuild and push the updated Docker image to your container registry. Make sure to update the version tag.

12. Update your deployment to use the new version of your Docker image. This can be done by editing the deployment or using kubectl set image.

### Part 4: Cleanup

### Delete the Deployment and Service:

13. Ensure you know how to clean up by deleting the deployment and service you created to avoid incurring any costs, especially if you are using a cloud provider.
14. Use kubectl delete deployment your_deployment_name and kubectl delete service your_service_name.
