# FROM node:18-alpine3.17

# WORKDIR /usr/app

# COPY package*.json /usr/app/

# RUN npm install

# COPY . .

# ENV MONGO_URI=uriPlaceholder
# ENV MONGO_USERNAME=usernamePlaceholder
# ENV MONGO_PASSWORD=passwordPlaceholder

# EXPOSE 3000

# CMD [ "npm", "start" ]

# Dockerfile
FROM node:18

# Optional: install additional tools here
# RUN apt-get update && apt-get install -y git

# Create a non-root user (optional but recommended)
RUN useradd -m jenkins
USER jenkins

ENV NPM_CONFIG_CACHE=/home/jenkins/.npm
RUN mkdir -p /home/jenkins/.npm && chown -R jenkins:jenkins /home/jenkins/.npm

WORKDIR /home/jenkins/app
