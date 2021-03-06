# Wordpress Configuration with **composer.json** and **Docker**

- Install docker on your testing server
- Clone repository
- Create _.env_ file (see sample)
- Build containers with docker compose. `docker-compose up --build`

## Creating the Wordpress Docker configuration

- My docker file is based on the official WordPress docker container
  - [Wordpress Docker Image](https://github.com/docker-library/wordpress)
  - [PHP Docker Image](https://github.com/docker-library/php)
- Uses Composer for configuration
  - [Wordpress Composer Config](https://composer.rarst.net)
- Using MariaDB for local development
  - [Docker Compose MariaDB](https://onexlab-io.medium.com/docker-compose-mariadb-5eb7a37426a2)

## Developer environment using DevContainers

- Docker installed
- Already configured in devcontainer
- Add ssh keys to allow git access

```sh
eval "$(ssh-agent -s)"
ssh-add ~/ssh/github_catenare
```

- Steps
  1. Start the dev container
  - **F1** _remote-container: Reopen container_ - Will restart with a devcontainer configuration
  1. Add local .env variables to container environment.
  - `set -a; source .env; set +a`
  <!-- 1. Copy _wp-config.php_ to _/var/www/html_ -->
  1. Run composer install. `composer install`
  <!-- - Will put wordpress in _/var/www/html/wordpress_ -->
  1. Run php built-in server. `php -S 0.0.0.0:8080 -t wordpress/
  1. Open your browser to _http://localhost:8000_

## Setting up Nginx Unit

- Custom build php does not make it easy to run unit

### Building custom docker image

- https://unit.nginx.org/installation/#initial-configuration

* https://www.docker.com/blog/multi-arch-images/

```sh
git clone https://github.com/nginx/unit
cd unit
git checkout 1.26.1
cd pkg/docker/
# make build-php7.4 VERSION_php=7.4
make dockerfiles VERSION_php=7.4
```

```sh
export UNIT=$(                                             \
      docker run -d --mount type=bind,src="$(pwd)/site",dst=/www  \
      -p 8080:8000 unit:1.27.0-php7.4               \
  )
docker exec -ti $UNIT curl -X PUT --data-binary @/www/config.json  \
      --unix-socket /var/run/control.unit.sock  \
      http://localhost/config
```

## Resources

- PHP
  - Using Microsoft PHP containers for devcontainer setup.
    - [Microsoft VSCode Dev Containers](https://github.com/Microsoft/vscode-dev-containers)
    - [php-mariadb container](https://github.com/microsoft/vscode-dev-containers/tree/v0.209.6/containers/php-mariadb)
    * [PHP Built-in Server](https://www.php.net/manual/en/features.commandline.webserver.php)
      - `php -S localhost:8000 -t wordpress/`
- Nginx Unit
  - nginx - https://unit.nginx.org/installation/?_ga=2.164349946.45652302.1588254197-1506587029.1585825834#installation-docker
  - php - https://unit.php.net/manual/en/installation.docker.html
- Install Docker on Ubuntu

  - https://docs.docker.com/engine/install/ubuntu/
  - https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user

  ### Task list

  - Composer file
  - php
  - Wordpress
  - MariaDB
  - Install nginx unit - https://unit.nginx.org

  ### Setup Wordpress with nginx unit

  - https://www.nginx.com/blog/automating-installation-wordpress-with-nginx-unit-on-ubuntu/
  - https://nucker.me/how-to-host-a-wordpress-site-in-docker-with-nginx/
  - https://appwrk.com/how-to-set-up-dockerizing-wordpress-with-nginx-web-server
  - Unit Wordpress - https://github.com/tippexs/unitwp
  - https://www.dmuth.org/wordpress-5-in-docker-with-nginx-and-letsencrypt/
  - https://medium.com/swlh/wordpress-deployment-with-nginx-php-fpm-and-mariadb-using-docker-compose-55f59e5c1a
  - https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-with-docker-compose

## Push image to Github registry

- docker/login-action:
  - https://github.com/docker/login-action#github-container-registry
  - https://github.com/marketplace/actions/docker-login
- docker/build-push-action
  - https://github.com/docker/build-push-action
  - https://github.com/marketplace/actions/docker-build-push-action

## Debugging Docker image

- `docker run --rm -it -p 8000:8000/tcp nziswano:wordpress bash`

## Github action to push to AWS Registry

- Need to push generated image to AWS registry

# CDK Configuration for our AWS Cloud Based WordPress instance

This is a project to automate the deployment of WordPress as a service into AWS.

- TODO: Put diagram of overall system architecture

## CDK Details

The `cdk.json` file tells the CDK Toolkit how to execute your app.

### Useful commands

- `npm run build` compile typescript to js
- `npm run watch` watch for changes and compile
- `npm run test` perform the jest unit tests
- `cdk deploy` deploy this stack to your default AWS account/region
- `cdk diff` compare deployed stack with current state
- `cdk synth` emits the synthesized CloudFormation template

## AWS ECR Config

- Container registry via CDK

* Created the `aws-ecr-repo` stack
* Added test to test against generated cloudformation

### Adding a github action to deploy this stack
