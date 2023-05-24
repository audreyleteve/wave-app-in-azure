<p align="center">
  <img src="https://github.com/audreyleteve/wave-app-in-azure/blob/main/static/h2oai_wave.png"  width="20%" height="20%"> <img src="https://github.com/audreyleteve/wave-app-in-azure/blob/main/static/Microsoft_Azure.png"  width="20%" height="20%">
</p>



H2O Wave is an open source software stack for building beautiful, low-latency, realtime, interactive, browser-based applications and dashboards entirely in Python without using HTML, Javascript, or CSS. H2O Wave makes it very easy to create data science applications, but it is even better when you can share it to the world, (or in your own [Microsoft Azure cloud][MS-Azure-Cloud] environment ;) ).

If you want to know how to get started and build your own H2O Wave app, you can start [here][wave-docs] !


The following is a walk through on how to deploy and run a containerized H2O Wave app with [Azure App Service][Azure App Service].

## Pre-requisites

Before we begin, please make sure you have these requirements covered.

- [Docker][docker-get-started]
- [Azure Account][azure-portal]
- [Azure Ressource Group][azure-portal]


## Useful but not required

- [Azure CLI][Azure-cli]
- [Visual Studio Code][VSC]
- [Azure Service Plan][azure-service-plan]


## Overview

1. Create a Wave app and make sure it is running on our local machine.
2. Dockerize the Wave app.
3. Deploy the Docker image to Azure.
4. Cake.

## Step by Step

Let's begin.


### 1. Create a Wave App

You can create your own Wave app ([examples][wave-examples]) or use an existing one. For the sake of this tutorial, let's clone from [wave-app-in-azure-repo] and use the provided app (developped using this [Wave tutorial][wave-hello]!)

If you already have an app ready to go, you can skip to the next section.

#### Clone the repo

```shell
$ git clone https://github.com/audreyleteve/wave-app-in-azure.git
```

#### Setup Virtual Environment

```shell
$ python -m venv .venv
$ . .venv/bin/activate
(.venv)$ pip install --upgrade pip
(.venv)$ pip install -r requirements.txt
```

#### Run the Wave App

Let's run our app on our local machine.
If you are using Wave version older than `0.20.0`, you need to start the Wave server (`waved`) before running the
Python app. Please refer to the [Wave Docs][wave-older-versions].

```shell
(.venv)$ wave run helloworld.py
 
2022/02/04 19:13:52 #
2022/02/04 19:13:52 # ┌────────────────┐ H2O Wave
2022/02/04 19:13:52 # │  ┐┌┐┐┌─┐┌ ┌┌─┐ │ 0.20.0 20220131055911
2022/02/04 19:13:52 # │  └┘└┘└─└└─┘└── │ © 2021 H2O.ai, Inc.
2022/02/04 19:13:52 # └────────────────┘
2022/02/04 19:13:52 #
```

The app will be available at http://localhost:10101 :

<img src="https://github.com/audreyleteve/wave-app-in-azure/blob/main/static/hello_world_app.png"  width="50%" height="50%">


### 2. Dockerize the Wave app

#### Get Dockerfile

For this step we need to add 3 new files to our repo. All three files are available in this repository[wave-app-in-azure-repo].
Place all three files at the root of the repo/project.

- [Dockerfile][wave-app-dockerfile] - This is the file we need to build our Docker image.
- [docker-entrypoint.sh][wave-app-docker-entrypoint] - This is the bash script that is called to start out Wave app inside the Docker container.
- [.dockerignore][wave-app-dockerignore] - This file is similar to `.gitignore` file which contains the list of all unnecessary files.

If you are following along from the top and cloned the example repo, the files are already there.

**Note**

Please make sure there is a `requirements.txt` file at the root of the repo/project. If
you don't have one, please generate one using the following command

While the virtual environment is activated, run

```shell
(.venv)$ pip freeze > requirements.txt
```

#### Build the Docker Image

Be at the root of the repo and run this command to create a Docker image with our Wave app inside it.
Change the values of `PYTHON_VERSION`, `WAVE_VERSION`, and `PYTHON_MODULE` to match your setup.
`PYTHON_MODULE` is same as the last part of the `wave run <PYTHON_MODULE>` command.

```shell
$ docker build --platform linux/x86_64 \
--build-arg PYTHON_VERSION=3.9.12 \
--build-arg WAVE_VERSION=0.25.3 \
--build-arg PYTHON_MODULE="helloworld.py" \
-t wave-helloworld:0.1.0 .
```

Check that our image is available

```shell
$ docker image ls
REPOSITORY                  TAG                IMAGE ID       CREATED         SIZE
wave-helloworld            0.1.0             c694d6f583b1     5 days ago     1.57GB
```

#### Run the Wave app using Docker

Let's start a Docker container from the Docker image we created.

**Note**:

In the following command, you can pick any `PORT` values for `-p` and `-e PORT`.
When running locally, we can select any value for `PORT` as long as it is available.
However, we need to use the same `PORT` value for `-p` option. Ex: `-p <app-port-outside-container>:<waved-port-inside-container>`
If we select our `PORT` inside the container to be `10101` and port outside the container to be `8080`, the command will look like
`-p 10101:8080 -e PORT=8080`. 

In this case, the app will be available at http://localhost:8080.

```shell
$ docker run --rm --name wave-helloworld -p 10101:8080 -e PORT=8080 wave-helloworld:0.1.0

$ ( cd /home/appuser/wave/wave-0.25.3-linux-amd64 && ./waved -listen ":8080" & )

params: []2022/05/25 16:04:28 # 
2022/05/25 16:04:28 # ┌────────────────┐ H2O Wave 
2022/05/25 16:04:28 # │  ┐┌┐┐┌─┐┌ ┌┌─┐ │ 0.21.1 20220512131551
2022/05/25 16:04:28 # │  └┘└┘└─└└─┘└── │ © 2021 H2O.ai, Inc.
2022/05/25 16:04:28 # └────────────────┘
2022/05/25 16:04:28 # 
2022/05/25 16:04:28 # {"address":":8080","base-url":"/","t":"listen","web-dir":"/home/appuser/wave/wave-0.21.1-linux-amd64/www"}

$ wave run --no-reload --no-autostart helloworld.py

2022/05/25 16:04:36 # 
2022/05/25 16:04:36 # ┌────────────────┐ H2O Wave 
2022/05/25 16:04:36 # │  ┐┌┐┐┌─┐┌ ┌┌─┐ │ 0.21.0 20220413151435
2022/05/25 16:04:36 # │  └┘└┘└─└└─┘└── │ © 2021 H2O.ai, Inc.
2022/05/25 16:04:36 # └────────────────┘
2022/05/25 16:04:36 # 
2022/05/25 16:04:36 # {"address":":10101","base-url":"/","t":"listen","web-dir":"/home/appuser/venv/www"}
2022/05/25 16:04:37 * /hello {"d":[{"k":"quote","d":{"view":"markdown","box":"1 1 2 2","title":"Hello World","content":"\"The Internet? Is that thing still around?\" - *Homer Simpson*"}}]}
INFO:     Started server process [1]
INFO:     Waiting for application startup.
INFO:     ASGI 'lifespan' protocol appears unsupported.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
```

Here, app will be available at http://localhost:10101



### 3. Deploy the Docker image to Microsoft Azure
Please note the below can be done also via Visual Studio code using Azure and Docker extensions or directly onto Azure portal.

### Log into Azure using CLI 

Once we have Azure CLI installed, we can login to Azure using the following command.
This will open a browser and ask you to login to Azure.

```shell
$ az login
```

### Log into Container Registry

**if you don't have already a container registry, create one using (skip otherwise):**

```shell
$ az acr create --name myazurecontainerregistry \
--resource-group myResourceGroup \
--sku Basic --admin-enabled true
$ az acr login -n myazurecontainerregistry
Login Succeeded
```

### Tag the Docker Image

Use the following command to give a new tag to our existing Docker image.

```shell
$ docker tag wave-helloworld:0.1.0 myazurecontainerregistry.azurecr.io/wave-helloworld:0.1.0
The push refers to repository [myazurecontainerregistry.azurecr.io/wave-helloworld]
[...]
0.1.0: digest: sha256:ab83ead854a741ab4d147cc2c4cc01557d9c3051d151d4e1bed674426f336fe5 size: 3256
```

### Push the image to Azure

Push our docker image to Azure's registry.

```shell
$docker push myazurecontainerregistry.azurecr.io/wave-helloworld:0.1.0
$az acr repository list -n myazurecontainerregistry
[
  "wave-helloworld"
]
$az acr repository show-tags -n myazurecontainerregistry --repository wave-helloworld
[
  "0.1.0"
]
```

### Release the image to your app
**if you don't already have an an App Service plan (skip otherwise):**
```shell
$ az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --is-linux
```

Now you created Docker images stored in your repository in Azure Container Registry. Use Azure App Service to run web apps that are based on the image you pushed in your Container Registry. Let's create the web app:

```shell
$ az webapp create --name myhelloworldwebapp \
--resource-group myResourceGroup \
--plan myAppServicePlan \
--deployment-container-image-name myazurecontainerregistry.azurecr.io/wave-helloworld:0.1.0

$ az webapp config appsettings set --resource-group myResourceGroup \
--name myhelloworldwebapp \
--settings PORT=8080
```


### Deploy your wave app on web app services and test it 

Finally, tell Azure to deploy your app and make it publicly available on the internet.

```shell
$az webapp config container set --name myhelloworldwebapp \
--resource-group myResourceGroup \
--docker-custom-image-name myazurecontainerregistry.azurecr.io/wave-helloworld:0.1.0 \
--docker-registry-server-url https://myazurecontainerregistry.azurecr.io
```
or update the version of the docker image on the web app's container settings :
```shell
$az webapp config container set --name myhelloworldwebapp \
--resource-group myResourceGroup \
--docker-custom-image-name myazurecontainerregistry.azurecr.io/wave-helloworld:0.2.0
```

To test the app, browse to https://myhelloworldwebapp.azurewebsites.net 
in our case, with the right route, it will be available at https://myhelloworldwebapp.azurewebsites.net/hello

Enjoy!


[docker-get-started]: https://www.docker.com/get-started

[docker-get-started]: https://www.docker.com/get-started
[azure-portal]: https://portal.azure.com/#home
[VSC]: https://code.visualstudio.com/
[Azure-cli]: https://docs.microsoft.com/en-us/cli/azure/install-azure-cli
[azure-service-plan]: https://docs.microsoft.com/en-us/azure/app-service/app-service-plan-manage
[MS-Azure-Cloud]: https://azure.microsoft.com/
[Azure App Service]:https://azure.microsoft.com/en-gb/products/app-service

[wave-discussions]: https://github.com/h2oai/wave/discussions
[wave-docs]: https://wave.h2o.ai/docs/getting-started
[wave-examples]: https://wave.h2o.ai/docs/examples
[wave-hello]: https://wave.h2o.ai/docs/tutorial-hello
[wave-issues]: https://github.com/h2oai/wave/issues
[wave-older-versions]: https://wave.h2o.ai/docs/installation-8-20
[wave]: https://wave.h2o.ai/

[wave-app-in-azure-repo]: https://github.com/audreyleteve/wave-app-in-azure.git
[wave-app-docker-entrypoint]: https://github.com/audreyleteve/wave-app-in-azure/docker-entrypoint.sh
[wave-app-dockerfile]:  https://github.com/audreyleteve/wave-app-in-azure/Dockerfile
[wave-app-dockerignore]:  https://github.com/audreyleteve/wave-app-in-azure/.dockerignore


