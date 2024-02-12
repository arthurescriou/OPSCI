# TME5 : GitLab CI/CD

This TME aims to familiarize you with CI/CD concepts and using GitLab Runner to automate the deployment of a application.

### Part 1: Setup

#### 1. Clone the project

Clone the following project:

```bash
git clone https://github.com/arthurescriou/redis-node.git
```

#### 2. Configure the GitLab remote

Modify the project's remote to point to your GitLab repository.

```bash
git remote set-url origin https://gitlab.com/<your_username>/redis-node.git
```



#### 3. Create a GitLab Runner with token access

Log in to GitLab:


```bash
gitlab login
```

Enter your GitLab username and password.

```bash
gitlab config set --global url your_gitlab_url
gitlab config set --global personal_access_token your_access_token
```


Replace ``your_gitlab_url`` with your GitLab instance URL (e.g., https://gitlab.com) and ``your_access_token`` with a personal access token with sufficient permissions (e.g., "api"). You can create a personal access token with the required scope from your GitLab profile settings.

Register the runner:


```bash
gitlab runner register \
  --url your_gitlab_url \
  --token your_access_token \
  --executor shell \
  --name "your_runner_name" \
  --description "Shell runner for CI/CD" \
  --tags "node"
```


Replace the placeholders with your values:
* ``your_gitlab_url``: Your GitLab instance URL
* ``your_access_token``: Your personal access token
* ``your_runner_name``: A descriptive name for your runner
* ``node``: An optional tag indicating your runner's capabilities (replace with other relevant tags if needed)

If successful, the command will output details about your newly registered runner.

Start the runner:

```bash
gitlab-runner run &
```

This will start the runner in the background, allowing it to execute jobs sent by your GitLab instance.


**Additional points:**

* You can use ``gitlab-runner`` info to check the status of your runner.
* You can use ``gitlab-runner`` unregister to unregister your runner.


#### 4.Create the pipeline

Create a ``.gitlab-ci.yml`` file at the root of the project.

Un exmple du contenue qu'on peut retrouver dans le fichier .yml:

```yml
image: node:16
stages:
  - build
  - test
  - deploy

build:
  script:
    - npm install
    - npm run build

test:
  script:
    - npm run test

deploy:
  script:
    - docker build -t <image_name> .
    - docker push <image_name>
    - ssh <user>@<server> "docker pull <image_name> && docker run -d <image_name>"
```

Replace the placeholders with your values:

* ``<image_name>``: The name of your image to build and deploy
* ``<user>@<server>``: The username and server address for where you want to deploy the image

### Part 2: Execution
#### 1.Register the runner:

```bash
gitlab-runner register
```

#### 2.Start the runner:

```bash
gitlab-runner run &
```

#### 3.Push your changes to GitLab:

```bash
git push
```
The pipeline will be automatically triggered.

### Part 3 :Adding a test

#### 1.Create a test script:

```bash
console.log("Hello world");
process.exit(0);
```

#### 2.Modify the .gitlab-ci.yml file:

```yml
test:
  script:
    - npm run test
    - npm run coverage
```

### Part 4 : Deploy to Docker Hub using :
#### 1.Create a Docker Hub account:

Visit https://hub.docker.com/ and sign up/sign in.

#### 2.Configure image name and authentication

In your .gitlab-ci.yml file, replace <image_name> with your desired Docker Hub image name (your_username/<image_name>).

Add a docker login step before the docker build and docker push steps:

```yml
...
deploy:
  image: docker/cli:latest
  script:
    - docker login -u your_dockerhub_username -p your_dockerhub_token
    - docker build -t your_username/<image_name> .
    - docker push your_username/<image_name>

```
Replace placeholders with your Docker Hub username and token:

```bash
your_dockerhub_username=your_actual_username
your_dockerhub_token=your_personal_access_token
```

#### 3.Push changes and trigger pipeline:

The pipeline will automatically run due to GitLab CI/CD's push triggers.

### Part 5 : GitLab Pages deployment using:

#### 1.Activate GitLab Pages:

* Go to your GitLab project settings -> Pages.
* Choose a subdomain or use your root domain.
* Click "Save changes."

#### 2.Add GitLab Pages job to .gitlab-ci.yml