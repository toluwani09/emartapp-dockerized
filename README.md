# Dockerized Microservices Deployment on AWS
# Project Overview

This project documents the transformation of a legacy monolithic Angular web application (developed some years ago) into a modern microservices-based architecture, fully containerized using Docker and deployed on AWS EC2.

The goal was to improve scalability, maintainability, and deployment efficiency, while also enabling faster developer onboarding and better collaboration between teams.

At the time of this documentation, the application is running in containers on AWS EC2.
# Note: HTTPS certificate setup and domain mapping to the EC2 public IP are pending and will be implemented later.

🏗️ Architecture

The application was split into independent services that communicate with each other:

Client Service (Angular) – built with Node.js and served via Nginx

Java API Service – backend REST API built with Java + Maven

Node.js API Service – handles business logic for specific features

Nginx Reverse Proxy – routes traffic across microservices

MongoDB – NoSQL database for application state

MySQL – relational database for structured data

All services are containerized with Dockerfiles and orchestrated using docker-compose.

High-level flow:

Client requests → Nginx → routed to APIs

APIs (Java + Node) → connect to MongoDB and MySQL

Client assets served from Angular build via Nginx

🛠️ Tools & Technologies

Docker & Docker Compose → containerization and orchestration

AWS EC2 (Ubuntu) → hosting the Docker engine and services

Nginx → reverse proxy and serving static client files

Java + Maven → build and package backend JAR

Node.js → build client assets and run API service

MongoDB + MySQL → persistent storage

GitHub → version control and deployment source

⚙️ Project Setup
1. Clone Repository
git clone <repo-url>
cd project-directory

2. Build Client (Angular)

The client service is built with Node.js and then served with Nginx.

Client Dockerfile (simplified):

FROM node:14 AS web-build
WORKDIR /usr/src/app
COPY ./ ./client
RUN cd client && npm install && npm run build --prod

FROM nginx:latest
COPY --from=web-build /usr/src/app/client/dist/client/ /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 4200

3. Build Java API

The Java API is built with Maven and packaged as a JAR file.

Java Dockerfile (simplified):

FROM openjdk:8 AS BUILD_IMAGE
WORKDIR /usr/src/app/
RUN apt update && apt install maven -y
COPY ./ /usr/src/app/
RUN mvn install -DskipTests

FROM openjdk:8
WORKDIR /usr/src/app/
COPY --from=BUILD_IMAGE /usr/src/app/target/book-work-0.0.1-SNAPSHOT.jar ./book-work-0.0.1.jar
EXPOSE 9000
ENTRYPOINT ["java","-jar","book-work-0.0.1.jar"]

4. Orchestrating with Docker Compose

All services are defined and orchestrated in a docker-compose.yaml file.

Key services include:

client (Angular + Nginx)

api (Node.js)

webapi (Java API)

nginx (reverse proxy)

emongo (MongoDB)

emartdb (MySQL)

Deployment command:

docker-compose build
docker-compose up -d

☁️ AWS EC2 Deployment

To host Docker and Docker Compose, an Ubuntu EC2 instance was launched on AWS.

Steps followed:

Created a new EC2 instance with Ubuntu AMI

Configured Security Group with inbound rules (HTTP, HTTPS, SSH, custom app ports)

Generated Key Pair (.pem) for SSH access

Added User Data script to auto-install Docker and Docker Compose:

#!/bin/bash
apt-get update -y
apt-get install -y docker.io docker-compose
usermod -aG docker ubuntu


Launched and SSH’d into the instance:

ssh -i keypair.pem ubuntu@<EC2-Public-IP>


Verified Docker setup:

docker-compose -version


Cloned GitHub repo to EC2:

git clone <repo-url>
cd project-directory


Built and deployed containers:

docker-compose build
docker-compose up -d

# Outcome

Deployment Time: Reduced from hours to minutes with docker-compose up

Scalability: Each microservice can now be scaled independently

Maintainability: Modular structure makes it easy to extend features or replace services

Collaboration: Developers can spin up the full stack locally with one command

# Next Steps

Add HTTPS certificates (via Let’s Encrypt or AWS ACM)

Map a domain name to the EC2 instance public IP

Integrate with CI/CD pipeline for automated builds and deployments

# Documentation

To make the project reproducible and transparent, screenshots and video recordings of the deployment process are included in the emartapp/screenshots and vid directory.

# Credits

Project guided and delivered by Tolulope Olalere – AWS Certified DevOps Engineer

Built with contributions from client dev team and open-source tools
