# Configuring a customized webpage for the webserver

## Prerequisites:
* Docker installed on your local machine
* Docker Hub account

# Signing into your docker hub account on your local machine:
Enter your username and then enter your password when prompted.
```
docker login --username=your-dockerhub-username
```


## Creating the customized webpage
Create an HTML file called `my-webpage.html` in the same directory as your other files for this project. The file should contain the following HTML code:
```
<!DOCTYPE html>
<html>
<head>
    <title>My Webpage</title>
    <style>
        body {
            background-color: #FF7F7F;
            font-family: Arial, sans-serif;
            text-align: center;
            padding-top: 20%;
        }
    </style>
</head>
<body>
    <h1>Hello, my name is .</h1>
</body>
    <p>Fun Fact</p>
</html>
```

## Creating a Dockerfile
Create a file called `Dockerfile` in the same directory as your other files for this project. The file should contain the following code:
```
FROM nginx:latest
COPY ./my-webpage.html /usr/share/nginx/html/index.html
```

## Building the Docker image
After creating the Dockerfile, build the Docker image using the following command:
```
docker build -t my-webserver:v1.0.0 .
```

## Running the Docker container
After building the Docker image, run the Docker container using the following command:
```
docker run -d -p 8080:80 --name my-custom-webpage my-webserver
```

## Testing the webpage
After running the Docker container, open a web browser and go to `http://localhost:8080` to see the webpage.

## Push the image to Docker Hub
After building the Docker image, tag the Docker image using the following command:
```
docker tag my-webserver:v1.0.0 your-dockerhub-username/my-webserver:v1.0.0
```

Then push the Docker image to Docker Hub using the following command:
```
docker push your-dockerhub-username/my-webserver:v1.0.0
```

## ECS
Confirn that that your docker image has been pushed to Docker Hub, you can now open the task definition file in ECS (use the one in this directory). 

Once there, replace the image name with your username/my-webserver. 
```
"image": "your-dockerhub-username/my-webserver",
```

This will allow ECS to pull your image from Docker Hub. After this step you can follow the previous instructions in the README.md file in the main directory. Remember to use the task definition file contained in this directory.





