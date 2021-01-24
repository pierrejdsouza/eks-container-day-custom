# Docker Basics

## Getting to now Docker
In this step, you check if Docker is installed on the AWS Cloud9 workspace, and download and run a standard container image of the popular web server Nginx.

1. Let’s start by running docker --version to confirm that both the client and server are there and working.
```
docker --version
```
2. Docker containers are built using images. Let’s run the command docker pull nginx:latest to pull down the latest nginx trusted image from Docker Hub.
```
docker pull nginx:latest
```
3. Now run docker images to verify that the image is now on your local machine’s Docker cache. If we start it then it won’t have to pull it down from Docker Hub first.
```
docker images
```
4. Now let’s try docker run -d -p 8080:80 --name nginx nginx:latest to instantiate the nginx image as a background daemon with port 8080 on the host forwarding through to port 80 within the container
```
docker run -d -p 8080:80 --name nginx nginx:latest
```
5. Run docker ps to see that our nginx container is running.
```
docker ps
```
6. Try curl http://localhost:8080 to use the nginx container and verify it is working with its default index.html.
```
curl http://localhost:8080
```
7. Running docker logs nginx shows us the logs produced by nginx and the container. You should see some events that correspond to our curl requests.
```
docker logs nginx
```
8. Use docker exec -it nginx /bin/bash to start an interactive shell into the container’s filesystem and constraints
```
docker exec -it nginx /bin/bash
```
9. From within the container, run cd /usr/share/nginx/html and cat index.html to see the content the nginx is serving which is part of the container.
```
cd /usr/share/nginx/html
cat index.html
```
10. Type exit to exit our shell within the container.
```
exit
```
11. Now run docker stop nginx to stop the container.
```
docker stop nginx
```
12. Try docker ps -a command and you should see that our container is still there but stopped. At this point it could be restarted with a docker start nginx if we wanted.
```
docker ps -a
```
13. To remove the container once and for all, use the docker rm nginx command.
```
docker rm nginx
```
14. Now you can use docker rmi nginx:latest to remove the nginx image from our machine’s local cache
```
docker rmi nginx:latest
```

## Building a container image

In this step, you use a Dockerfile to build a container image onto the instance. This sample uses an image that includes the Nginx webserver to serve a simple html page.

1. Let’s start by creating a working directory for our container image
```
 mkdir ~/environment/container-image
```
2. Type cd container-image to change into that folder.
```
cd ~/environment/container-image
```
3. Run touch Dockerfile to create a blank Dockerfile. This file will contain a set of steps required to build out container image.
```
touch Dockerfile
```
4. Add the following contents to the Dockerfile file
```
cat <<EOF > Dockerfile
FROM nginx:latest
COPY index.html /usr/share/nginx/html
EOF
```
5. Run touch index.html to create a blank html file which will contain a simple message.
```
touch index.html
```
6. Use the echo command line tool to pipe a simple message in to our index.html file.
```
echo "We've added our own custom content into the container" >> index.html
```
7. Use docker build -t nginx:1.0 . to build the nginx container image from our Dockerfile
```
docker build -t nginx:1.0 .
```
8. You can now use docker history nginx:1.0 to see all the steps and base containers that our nginx:1.0 is built on. Note that our change amounts to one new tiny layer on top.
```
docker history nginx:1.0
```
9. Type docker run -p 8080:80 --name nginx nginx:1.0 to run our new container. Note that we didn’t specify the -d to make it a daemon which means it holds control of our terminal and outputs the containers logs to there which can be handy in debugging.
```
docker run -p 8080:80 --name nginx nginx:1.0
```
10. Open another Terminal tab (Window -> New Terminal)
11. Run curl http://localhost:8080 in the other tab a few times and see our new content.
```
curl http://localhost:8080
```
12. Go back to the first tab and see the log lines sent right out to STDOUT.
13. At this point we could push it to Docker Hub or a private registry like Amazon ECR for others to pull and run. You will learn more about that shortly.
14. Type Ctrl-C to exit the log output. Note that the container has been stopped but is still there by running a docker ps -a.
```
docker ps -a
```
15. Use sudo docker inspect nginx to see lots of info about our stopped container.
```
sudo docker inspect nginx
```
16. As we did earlier, use docker rm nginx to delete our container.
```
docker rm nginx
```
17. For our last magic trick, we’re going to try mouting some files from the host into the container rather than embedding them in the image. Run docker run -d -p 8080:80 -v /home/ec2-user/environment/container-image/index.html:/usr/share/nginx/html/index.html:ro --name nginx nginx:latest
```
docker run -d -p 8080:80 -v /home/ec2-user/environment/container-image/index.html:/usr/share/nginx/html/index.html:ro --name nginx nginx:latest
```
18. Now try a curl http://localhost:8080. Note that even though this is the upstream nginx image from Docker Hub our content is there.
```
curl http://localhost:8080
```
19. Edit the index.html file and add some more things.
```
echo "This is another line I've added to my container" >> index.html
```
20. Try another curl http://localhost:8080 and note the immediate changes.
```
curl http://localhost:8080
```
21. Finally, run docker stop nginx and docker rm nginx to stop and remove our last container.
```
docker stop nginx && docker rm nginx
```
