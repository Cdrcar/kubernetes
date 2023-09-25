### 1. Creating Docker File

In order to work with Docker you will first need it installed on your own computer

Navigate to [Docker.com](https://www.docker.com/) in order to install Docker Desktop

2. The Docker Hub has an official Node.js Docker image to run react app (node:14.16.1-alpine3.13)

3. Building the front end container by creating file called Dockerfile in the root dirctory where the .json files exist and add to it this contents

```
# Use an official Node.js runtime as a parent image
FROM node:14.16.1-alpine3.13 AS base

# Set the working directory to /app
WORKDIR /app

# Copy the package.json and package-lock.json files
COPY ["package.json", "package-lock.json*", "./"]

# Install dependencies
RUN npm install

# Copy the rest of the application code
COPY . .

# for build

FROM base as builder

WORKDIR /app


# Build the application
RUN npm run build

# Expose port 8080
EXPOSE 5173

# Run the application
CMD ["npm", "start"]

```

4. Navigate to the Dockerfile and this command

```
docker build -t frontend:1.0 .
```

As long as you are within the directory that contains the Dockerfile (the . at the end of the command means "look in my current directory for the dockerfile) then the Docker build will kick in to action.

Then run this command to look at your docker images

```
docker images
```

5. To run the container locally run this command and map the container port **5173** with any unused port in your locallhost

```
docker run -p 5173:5173 frontend:1.0
```

### 2. CircleCI Pipline

1. Create CircelCI account

2. Create folder in your porject called .circleci and create file inside this folder called config.yml

3. Set up the config file to build the application and then run the tests you can explore the file within the .circleci directory.

4. Whilst within CircleCI navigate to the Projects section.

Filter the projects to show the frontend project and click the Set up project button.

In the resulting dialog, type main for the branch name and choose the Fastest option so that it automatically picks up your config.yml file.

This will then begin to build the application.

Let the build complete and it should hopefully go green 🤞 and look similar to the image below.

Go ahead and explore CircleCI and click into your build to see the steps

5. Setup CircleCI environment variables
   Next you will use CircleCI environment variables to configure CircleCI to use your AWS access key and secret access key.

In the CircleCI screen, navigate to the project and click the Project Settings button in the top right corner.

Then click Environment Variables

You are going to setup three environment variables as shown in the screenshot

Name Value
AWS_ACCESS_KEY_ID This should be the access key as per the user you created in the previous step
AWS_SECRET_ACCESS_KEY This should be the secret access key as per the user you created in the previous step
AWS_ECR_REGISTRY_ID This should be your 12 digit account ID (without the hyphens) and can be obtained by clicking your username top right corner of the AWS console

6. Enhance the project with image production
   Now you will update the config.yml to add a further stage for it to build a docker image and push to your container registry.

Open the config.yml file ready for editing.

Firstly you need to put in place the orb, beneath the version add in the orb:

```
orbs:
  aws-ecr: circleci/aws-ecr@8.2.1
```

Then within the jobs section, introduce a new job called build-image-and-push. This job will perform the steps for building the docker image, authenticating with your AWS account and pushing it up to your container registry.

Replace the REPLACE_ME_YOUR_REPOSITORY_NAME with the name you gave your repository. You can see your repository name listed on the under the Repository name column on the AWS console.

Replace the REPLACE_ME_YOUR_PUBLIC_REGISTRY_ALIAS with your Default alias. You can find the default alias by clicking the Public registry link on the left hand side navigation.

````
build-image-and-push:
    working_directory: ~/ce-cicd-sample-java-web
    docker:
      - image: cimg/aws:2023.05
    steps:
      - setup_remote_docker # Ensures that we can issue docker build commands
      - aws-ecr/build-and-push-image:
          repo: REPLACE_ME_YOUR_REPOSITORY_NAME
          path: "demo-app" # This is the directory to look for the Dockerfile
          build-path: "demo-app" # This is directory for Docker to load the build path to source code
          tag: ${CIRCLE_BUILD_NUM} # This will be automatically replaced with the job number
          public-registry-alias: REPLACE_ME_YOUR_PUBLIC_REGISTRY_ALIAS
          public-registry: true

          ```
Finally add the job to your workflow

````

workflows:
demo-app-pipeline:
jobs: - build-and-test - build-image-and-push:
requires: - build-and-test

            ```

You can see a fully complete example config in the finished_example.yml file.

6. Git commit and push your built image to its AWS ECR repository
