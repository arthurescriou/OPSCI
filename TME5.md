# TME5 : GitLab CI/CD

This TME aims to familiarize you with CI/CD concepts and using GitLab Runner to automate the deployment of a application.

### Part 1: Setup

#### 1. Clone the project

Clone the following project:

```bash
git clone https://github.com/arthurescriou/redis-node.git
```

or if you already have the repository

```bash
git pull origin master
```

#### 2. Configure the GitLab remote

Modify the project's remote to point to your GitLab repository.

```bash
git remote set-url origin https://gitlab.com/<yourname>/redis-node.git
```

#### 3. Create a GitLab Runner with token access

Login to gitlab.com. Create a repository `redis-node`.

Create a folder `/srv/gitlab-runner/config`.

```sh
mkdir -p /srv/gitlab-runner/config
```

Register the runner:

```bash
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```

You will need to specify some values:

- Your GitLab instance URL : `gitlab.com`
- A runner identification token ( you can generate one on this page : `https://gitlab.com/${yourname}/redis-node/-/runners/new`)
- A descriptive name for your runner
- A base image for your runner : `node:18-alpine` for instance

If successful, the container will start and you will be able to find it in the gitlab interface.

This will start the runner in the background, allowing it to execute jobs sent by your GitLab instance.

If the container runner logs print error like `gitlab-runner/config.toml no such file or directory`

Edit directly your `config.toml` file.

```toml
[[runners]]
  name = "node:18"
  url = "https://gitlab.com/"
  token = "<TOKEN>"
  limit = 0
  executor = "docker"
  builds_dir = ""
  shell = ""
  environment = ["ENV=value", "LC_ALL=en_US.UTF-8"]
  clone_url = "http://gitlab.example.local"
```

### Part 2 : Pipeline

#### 4.Create the pipeline

Create a `.gitlab-ci.yml` file at the root of the project :

```yml
image: node:18
stages:
  - build
  - test

build:
  script:
    - npm install

test:
  script:
    - npm run test
```

#### Push your changes to GitLab:

```bash
git push
```

The pipeline will be automatically triggered.

And if everything goes well you will see a success execution.

<img src="img/ciok.png"/>

### Part 3 : Add docker in the pipeline

#### 1.Create a Docker Hub account:

Visit https://hub.docker.com/ and sign up/sign in.

#### 2. Build a docker image in the pipeline

Add a job to build a docker image. (You will also need a `Dockerfile` at the root of the repository.)

```yml
docker:
  image: docker:cli
  script:
    - docker build -t <image_name> .
  rules:
    - if: $CI_COMMIT_BRANCH
      exists:
        - Dockerfile
```

Replace the placeholder with your value:

- `<image_name>`: The name of your image to build and deploy

#### 3.Configure image name and authentication

We want now to push our images on docker hube automatically.

Add a docker login step before the docker build and docker push step.

We also need to specify some variables in the secret environment of the repository :

- `CI_REGISTRY_USERNAME` : your dockerhub username
- `CI_REGISTRY_IMAGE` : the name of the image
- `CI_IMAGE_TAG` : the version of the image
- `CI_REGISTRY_PASSWORD` : your dockerhub password (be sure to put this one in hidden mode)

Add these configuration globally in your `gitlab-ci.yml`.

```yml
variables:
  DOCKER_IMAGE_NAME: $CI_REGISTRY_USERNAME/$CI_REGISTRY_IMAGE:$CI_IMAGE_TAG
before_script:
  - docker login -u "$CI_REGISTRY_USERNAME" -p "$CI_REGISTRY_PASSWORD"
```

And add a new step in your `docker` job.

```yml
- docker push "$DOCKER_IMAGE_NAME"
```

#### 4.Push changes and trigger pipeline:

The pipeline will automatically run due to GitLab CI/CD's push triggers.

### Part 5 : GitLab Pages deployment using:

Clone the react-redis repository and host it with gitlab pages.

#### 1.Activate GitLab Pages:

- Go to your GitLab project settings -> Pages.
- Choose a subdomain or use your root domain.
- Click "Save changes."

#### 2.Create a gitlab-ci.yml file

The gitlab-ci.yml must specify a job that will build the react project and will specify the `build` folder as artifact for gitlab pages.

### Part 6 : Deployment (bonus)

Now that every images are built. How can we deploy it on a kubernetes cluster or on machine with docker ? (from gitlab of course)
