 # Cowsay Dockerized Pipeline

This project implements a Jenkins pipeline to automate the versioning, building, and deployment of a Dockerized Cowsay application. The pipeline is triggered by changes in the Git repository and supports versioning based on release branches.

## Project Structure

- **`Jenkinsfile`**: The Jenkins pipeline script defining the stages and steps of the CI/CD process.
- **`Dockerfile`**: Dockerfile for building the Cowsay Docker image.
- **`v.txt`**: Text file containing the version information.
- **`gitlab-ssh`**: Jenkins credential for SSH access to GitLab.
- **`ssh-to-ec2`**: Jenkins credential for SSH access to EC2.

## Pipeline Stages

### 1. Git

- Checks out the Git repository.

### 2. Existence of the Branch

- Determines the existence of the release branch based on the provided version parameter.
- Creates a new branch if it doesn't exist.
- Updates the `v.txt` file with the new version number.
- Commits and pushes changes to the repository.

### 3. Calculate Last Version

- Reads the version from the `v.txt` file.
- Determines the latest Git tag matching the version.
- Updates the version in the `v.txt` file and increments the minor version if no matching tags are found.

### 4. Build

- Builds the Docker image using the Dockerfile.
- Tags the image with the updated version.

### 5. Run

- Runs the Docker container in detached mode on a specified network.

### 6. Test

- Conducts unit tests, such as checking the container's response and stopping the container.

### 7. Release

- Tags the Docker image for release in Amazon ECR.
- Pushes the tagged image to the ECR repository.

### 8. Git Push Tag

- Restores the `v.txt` file to its original state.
- Tags the Git repository with the updated version.
- Pushes the new tag to the GitLab repository.

## Pipeline Notifications

- Sends success or failure emails upon completion of the pipeline stages.

## Pipeline Usage

1. **Set Up Jenkins Credentials:**
   - Add a private key credential named `gitlab-ssh` for GitLab access.
   - Add a private key credential named `ssh-to-ec2` for EC2 access.

2. **Configure Jenkins Pipeline:**
   - Create a new pipeline job in Jenkins and point it to this Git repository.
   - Configure the pipeline with the necessary parameters and credentials.

3. **Trigger Pipeline:**
   - Manually trigger the pipeline or set up webhooks to trigger it automatically on GitLab changes.

4. **Review Pipeline Output:**
   - Monitor the Jenkins console output for progress and any error messages.

This pipeline automates the versioning, building, and deployment process for the Cowsay application. Adjust parameters and configurations as needed for your specific environment.
