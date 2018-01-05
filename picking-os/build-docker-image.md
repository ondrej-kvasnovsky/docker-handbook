# Build Docker Image

We are going to make git repository on our local staging server and we will automate way the new version is deployed there.

### Git Hook

When developer pushed code, the hook will checkout the code. Then it will build and tags docker image. When that is done, it restarts the servers.

Lets create a git repository on the staging server. And add a hook that will rebuild the docker container.

```
#!/usr/bin/env bash
SSH_USER="ondrej"
SERVER_IP="192.168.1.199"

echo "Initialize git repo and hooks..."
scp "git/post-receive/mobydock" "${SSH_USER}@${SERVER_IP}:/tmp/mobydock"
ssh -t "${SSH_USER}@${SERVER_IP}" bash -c "'
sudo apt-get update && sudo apt-get install -y -q git
sudo rm -rf /var/git/mobydock.git /var/git/mobydock
sudo mkdir -p /var/git/mobydock.git /var/git/mobydock
sudo git --git-dir=/var/git/mobydock.git --bare init

sudo mv /tmp/mobydock /var/git/mobydock.git/hooks/post-receive
sudo chmod +x /var/git/mobydock.git/hooks/post-receive
sudo chown ${SSH_USER}:${SSH_USER} -R /var/git/mobydock.git /var/git/mobydock
'"
echo "done!"
```

Here is the git hook that will redeploy the application.

```
#!/usr/bin/env bash
SSH_USER="ondrej"
SERVER_IP="192.168.1.199"
REPO_NAME="mobydock"

# Check out the newest version of the code.
export GIT_WORK_TREE="/var/git/${REPO_NAME}"
git checkout -f

TAG="$(git log --pretty=format:'%h' -n 1)"
FULL_COMMIT_TAG="${REPO_NAME}:${TAG}"
FULL_LATEST_TAG="${REPO_NAME}:latest"

# Build the image with the proper commit tag.
docker build -t "${FULL_LATEST_TAG}" "${GIT_WORK_TREE}"

# Get the Docker ID of the last built image.
DOCKER_ID="$(docker images -q ${REPO_NAME} | head -1)"

# Tag a latest version based off a commit tag.
#docker tag -f "${DOCKER_ID}" "${FULL_LATEST_TAG}"

echo "Restarting ${REPO_NAME}"
docker stop "${REPO_NAME}"

echo "Removing untagged Docker images (may take a while)"
docker rmi $(docker images --quiet --filter "dangling=true")
```

Add remote for staging into git.

```
git remote add staging ssh://ondrej@192.168.1.199:/var/git/mobydock.git
```



