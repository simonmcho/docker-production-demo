# Frontend for production-grade docker demo

## Instructions
• Using `Dockerfile.dev` for development startup    
• Run `docker build -f Dockerfile.dev .` in the directory to build the image    
• Run `docker run -it -p 3000:3000 CONTAINER_ID`    

This runs into an issue where updated code won't be reflected in the running container since the docker build takes a snapshot of the source code.
In order to have a straight copy, we need a different flow using `Docker volume`

### Docker Volumes

Instead of this:
```
Local Folder
  - frontend
  - /src
  - /public
```

Being copied over as 

```
Docker Container
  - /app
    - /src
    - /public
```

We will use it as:
```
Docker Container
  - /app
    - /src (references /src in local folder)
    - /public (references /public in local folder)
```

So we are essentially mapping references to the files. Sometimes the docker volumes are harder to setup, so we don't do this with simpler projects.

The essential "formula" for the docker volume command is:
`docker run -it -p 3000:3000 -v /app/node_modules -v $(pwd):/app <image_id>`    
• `-v /app/node_modules`: put a bookmark on the `node_modules` folder      
• `-v $(pwd):/app`: map the `pwd` into the `/app` folder      

If you try to run `docker run -it -p 3000:3000 -v $(pwd):/app <image_id>`, it will not work if you do not have `node_modules` directory locally. This is because `$(pwd)` is the current local working directory is the being referenced by what's on the right-hand side of `:`, which is `/app`: 

```
Docker Container
  - /app
    - /src (references /src in local folder)
    - /public (references /public in local folder)
    - /node_modules (references /node_modules in local folder) // THIS DOESN'T EXIST IN LOCAL FOLDER, SO HAS A NULL REFERENCE
```
So if you don't have `node_modules` locally, then you must add `-v /app/node_modules` as part of the command. This is saying that specific directory structure in the container should not be referencing anything in the local directory.

### Using docker-compose file for docker volumes
See `/docker-compose.yml` file for example.    
An update to `cra` leads to application exiting with code 0. Add stdin_open property to your docker-compose.yml file:
```
  web:
    stdin_open: true
```
Make sure you rebuild your containers after making this change with  docker-compose down && docker-compose up --build

https://github.com/facebook/create-react-app/issues/8688    
https://stackoverflow.com/questions/60790696/react-scripts-start-exiting-in-docker-foreground-cmd    

### Using dockerfile in docker-compose.yml
In our `Dockerfile.dev`, we have a command to copy everything in the pwd over to the docker image instance. But since we are using docker volumes and referencing directories from the container to the local folder (except `node_modules`), we don't need the copy command in the `Dockerfile.dev` file.

But, it's still worth having it because:
1. It reminds the devops to copy over the folders if `docker-compose.yml` is no longer being used
2. No more changes are needed in the pwd, especially if it's on production
