# What we are about to do?

1. Containerize the ChinookAPI project
2. Build the container image
3. Run the container image
4. Guest to Host Port mapping
5. Push the container image to docker hub

## 1. Containerize the ChinookAPI project

1. Open the ChinookAPI project in visual studio
2. Right click on the project and choose Add -> Add Docker Support as shown below

![Imgur](https://i.imgur.com/xFFeinJ.png)

3. The above action will create a __Dockerfile__ and a __docker-compose__ project

### Dockerfile

The __Dockerfile__ stores the configurations on how to build the container image. It is much similar to a bash script except there are some custom commands that docker can interpret to do specific operations as shown below.

```dockerfile
# Pull the ASP.NET core runtime image for running the Web API
FROM microsoft/dotnet:2.1-aspnetcore-runtime AS base
# Change the directory to /app in the container
WORKDIR /app
# Expose the port 80 of the container
EXPOSE 80
# Pull the ASP.NET core SDK for building the Web API
FROM microsoft/dotnet:2.1-sdk AS build
# Change the directory to /src in the container
WORKDIR /src
# Copy the ChinookAPI.csproj from the host to the container
COPY ChinookAPI/ChinookAPI.csproj ChinookAPI/
# Restore the nuget packages for the project in the container
RUN dotnet restore ChinookAPI/ChinookAPI.csproj
# Copy all the files from host to the container
COPY . .
# Change the directory to /src/ChinookAPI in the container
WORKDIR /src/ChinookAPI
# Build the ChinookAPI project using .NET core SDK tools
RUN dotnet build ChinookAPI.csproj -c Release -o /app

FROM build AS publish
# Publish the build artifacts to the /app directory in the container
RUN dotnet publish ChinookAPI.csproj -c Release -o /app

FROM base AS final
# Change the directory to /app in the container
WORKDIR /app
# Copy the build artifacts from the .NET SDK container to .NET runtime container
COPY --from=publish /app .
# Whenever the container starts run the command dotnet ChinookAPI.dll
ENTRYPOINT ["dotnet", "ChinookAPI.dll"]
```
### docker-compose.yml

The __docker-compose.yml__ stores configurations for orchestrating multiple container services using __Docker Swarm__. It is mostly used in development to spawn multiple container services.

```yml
version: '3.4'
services:
  # List of container services to start
  chinookapi:
    # Image of the container
    image: ${DOCKER_REGISTRY}chinookapi
    # Configuration for building the container image
    build:
      context: .
      # Path of the Dockcerfile for the container service
      dockerfile: ChinookAPI/Dockerfile
```

![Imgur](https://i.imgur.com/eUHvO5s.png)

## 2. Build the container image

1. Open a terminal in the root path of the project
2. Run the following command to build the image

```sh
docker-compose build
```

You should see something much similar as shown below

![Imgur](https://i.imgur.com/wm23m6C.png)

## 3. Run the container image

1. Open a terminal in the root path of the project
2. Run the following command to run the application within the container

```sh
docker-compose up
```

![Imgur](https://i.imgur.com/DavtPsm.png)

## 4. Guest to Host Port mapping

If you now try to access the API it won't be accessible, this is because by default docker does not expose any port from the container to the host machine. To expose a port from the container to the host machine we should add the following line to the __docker-compose.yml__ file

```yml
version: '3.4'
services:
  chinookapi:
    image: ${DOCKER_REGISTRY}chinookapi
    ports:
      - 5000:80
    build:
      context: .
      dockerfile: ChinookAPI/Dockerfile
```

Now you should be able to access the API at __http://localhost:5000__

You can also run the container from visual studio by setting the __docker-compose__ project as startup project.

![Imgur](https://i.imgur.com/8XzgKGi.png)

## 5. Pushing the image to docker hub

1. Create an account in ![docker hub](https://hub.docker.com/)
2. Login to the docker hub account using the command

```sh
docker login --username=yourhubusername --email=youremail@company.com
```

3. Push the image using the command

```sh
docker push yourhubusername/chinookapi:latest
```

Now you can download and run your docker image even from top of a mountain using the commands

```
docker pull yourhubusername/chinookapi:latest
docker run chinookapi -p 5000:80
```

