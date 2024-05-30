# Use an official Node.js runtime as the base image
FROM node:alpine

# Set the working directory inside the container
WORKDIR /usr/src/app

# Copy package.json and package-lock.json to the container
COPY package*.json ./

# Install dependencies inside the container
RUN npm install

# Rebuild bcrypt
# RUN npm rebuild bcrypt --build-from-source

# Copy the rest of the code into the container
COPY . .

# # Set the environment variable for Node.js
# ENV NODE_ENV=production

# Specify the command to run when the container starts
CMD [ "npm", "start" ]
