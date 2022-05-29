# [Install docker compose](https://docs.docker.com/compose/install/)
 - [x] pull from github `sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`
 - [x] Make executable ` sudo chmod +x /usr/local/bin/docker-compose`

# Run docker-compose.yaml
`docker-compose -f ./docker-compose.yaml up -d`
  - `-f` use file
  - `-d` run in detached mode (closing your session won't close the process)
