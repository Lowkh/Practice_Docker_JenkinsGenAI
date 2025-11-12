# Container and Orchestration
---

## Exercises

### Session Activity: Install Docker

Follow the instructions in the summary below to set up Docker if you have never used it before. 

---

## Jenkins in Docker: Beginner-Friendly Guide

### Step 1: Install Docker

Download Docker from [Docker's official site](https://docs.docker.com/get-docker/). Install Docker Desktop (Windows/Mac) or follow the Linux instructions from the Docker documentation. Confirm installation by running:

```bash
docker --version
```

### Step 2: Download and Run Jenkins in Docker

- Create a Docker bridge network for Jenkins:

```bash
docker network create jenkins
```

- Run Docker-in-Docker (DinD) container (if advanced builds needed):

```bash
docker run --name jenkins-docker --rm --detach \
  --privileged --network jenkins --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 docker:dind
```

    DinD is useful for advanced pipelines. Beginners may skip this if only basic functions are needed.

- Pull and start Jenkins container (Blue Ocean provides an enhanced UI):

```bash
docker run --name jenkins-blueocean --restart=on-failure --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  jenkins/jenkins:latest
```

    - `--publish 8080:8080` exposes Jenkins web interface on localhost:8080
    - `--publish 50000:50000` used for Jenkins agents
    - `--volume ...` ensures Jenkins data persists on your host

### Step 3: Unlock Jenkins and Basic Configuration

1. Access Jenkins by visiting http://localhost:8080 in a web browser.
2. Jenkins will prompt for an initial admin password.
3. Retrieve password using:

```bash
docker exec jenkins-blueocean cat /var/jenkins_home/secrets/initialAdminPassword
```

4. Copy and paste this into the browser prompt, then continue. Follow the guided setup wizard to create your admin account and install recommended plugins.

### Step 4: Install Git and GitHub Plugins in Jenkins

1. Go to **Manage Jenkins > Manage Plugins**.
2. Search for `Git` and `GitHub` plugins, then install.
3. Restart Jenkins if prompted.

### Step 5: Setting Up GitHub Personal Access Token

1. Go to your [GitHub settings](https://github.com/settings/tokens) > Developer Settings > Personal access tokens > Create new token (classic).
2. Give the token a name and required permissions (at minimum: `repo`).
3. Copy the token. Keep it secure: it grants access to your code.

### Step 6: Adding GitHub Credentials in Jenkins

1. In Jenkins UI: **Manage Jenkins > Manage Credentials > (Global) > Add Credentials**
2. Kind: "Username with password". Username: your GitHub username. Password: the personal access token.
3. Optionally, set an ID (like `github-access-token`). Save.

### Step 7: Connect Jenkins Job to GitHub Repository

1. Go to **New Item** to create a project (choose "Freestyle Project" for beginners).
2. Under **Source Code Management**, choose `Git`.
3. Paste your GitHub repository HTTPS URL (e.g., `https://github.com/yourusername/yourrepo.git`).
4. Select the credentials just created.
5. Set up a basic build step, like `echo Hello World!` in the "Build" section for testing. Save.

### Step 8: Set Up Jenkins Build Triggers from GitHub Webhook

1. On Jenkins job page, select "Configure".
2. Under "Build Triggers", check `GitHub hook trigger for GITScm polling`. Save.
3. In your GitHub repo, go to **Settings > Webhooks > Add webhook**.
4. Payload URL: `http://localhost:8080/github-webhook/` (for your local Jenkins, or use public IP/domain if remote).
5. Content type: `application/json`. Leave secret empty. Select "Just the push event".
6. Click "Add webhook".

Now Jenkins will automatically build when you push to GitHub.

---

## Troubleshooting Notes and Tips

- **Initial Jenkins password location**: `/var/jenkins_home/secrets/initialAdminPassword` (retrieve via `docker exec` command).
- **All Jenkins data is stored in mapped Docker volumes** – if you destroy the container but keep the volume, your data persists.
- **Jenkins can run basic builds without DinD** – DinD is only needed for advanced Docker steps inside build pipelines.

---

## Example: Simple Jenkinsfile for GitHub Integration

For a pipeline job using a Jenkinsfile:

```groovy
pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', credentialsId: 'github-access-token', url: 'https://github.com/yourusername/yourrepo.git'
      }
    }
    stage('Build') {
      steps {
        sh 'echo Build started!'
      }
    }
  }
}
```

---

## Summary

This expanded guide enables a beginner to install Docker, run Jenkins locally in Docker, unlock Jenkins, install all necessary plugins, securely connect to GitHub, and set up builds that trigger automatically each time code is pushed. All instructions use CLI and UI steps, reference real credentials setup, and provide troubleshooting advice for common roadblocks.
