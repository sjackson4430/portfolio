---
title: 'Create Docker Image'
date: 2022-01-29 00:00:00
description: Steve Jackson - Portfolio Website
featured_image: '/images/demo/demo-square.jpg'
---
<h1>
<a id="user-content-h1" class="anchor" href="#h1" aria-hidden="true"><svg class="octicon octicon-link" viewBox="0 0 16 16" version="1.1" width="16" height="16" aria-hidden="true"><path fill-rule="evenodd" d="M7.775 3.275a.75.75 0 001.06 1.06l1.25-1.25a2 2 0 112.83 2.83l-2.5 2.5a2 2 0 01-2.83 0 .75.75 0 00-1.06 1.06 3.5 3.5 0 004.95 0l2.5-2.5a3.5 3.5 0 00-4.95-4.95l-1.25 1.25zm-4.69 9.64a2 2 0 010-2.83l2.5-2.5a2 2 0 012.83 0 .75.75 0 001.06-1.06 3.5 3.5 0 00-4.95 0l-2.5 2.5a3.5 3.5 0 004.95 4.95l1.25-1.25a.75.75 0 00-1.06-1.06l-1.25 1.25a2 2 0 01-2.83 0z"></path></svg></a>Building python flask framework</h1>
Create your own Docker image for ease of deployment
Building python flask framework
1. os – Ubuntu
2. Update apt repo
3. Install dependencies using apt
4. Install Python dependencies using pip
5. Copy source code to /opt folder
6. Run the web server using "flask" command

Create the Dockerfile
This file contains a list of Instructions, followed by an Argument, software, dependencies, update, also copying our code to /opt/source-code location, and entrypoint and always starts with a Base From Image and entrypoint allows us to run a command when the image will be run in a container

$From Ubuntu

$RUN apt get update
$RUN apt-get install python
$RUN pip install flask
$RUN pip install flask-mysql

$Copy . /opt/source-code

$ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run



Then run this command to create an image locally on your system
docker build Dockerfile -t sjackson916/my-custom-app

To make it avilable on the docker registry use thie push command
docker push sjackson916/my-custom-app

Docker has layered archetecture
Layer 1 is the Base os
Layer 2 Changes in apt packages
Layer 3 Changes in pip packages
Layer 4 Copy the Source Code over
Layer 5 The entrypoint

you will see each layer contains a different amount of space
You will see if you run the docker history "Image Name" command
When running Docker Build, it will re-run layers from cache so the build will be quicker and the layers you made changes to or failed will be rebuilt.

So if your just updating the source code layer, just that layer of the build will be rebuilt and the rest will be cached which makes deployment of the docker image much faster.

You can containerize everything, skype, spotify, curl, firefox, chrome, etc...

<h2>Creating a New Docker Image
</h2>