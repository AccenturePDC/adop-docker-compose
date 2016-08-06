# To be improved! Current README is only intended for audience with docker and ADOP knowledge.

## Launching from the Commandline

- Export Container global variables.

    - `CUSTOM_NETWORK_NAME` - The docker network name.
    - `TARGET_HOST` - Use the host internal ip address where the proxy is deployed.
    - `LOGSTASH_HOST` - Use the host internal ip address where the proxy is deployed.
    - `INITIAL_ADMIN_USER` - The initial user that will have an administrator access to all the tools.
    - `INITIAL_ADMIN_PASSWORD_PLAIN` - The initial user's password.
    - `ADOP_CLI_USER` - The cli user that has administrator access to Jenkins. This can be the `INITIAL_ADMIN_USER`
    - `ADOP_CLI_PASSWORD` - The cli user's password.
    
    Or better create a script for it.
    
    ```bash
    cat > my-vars.sh <<-EOF
    export CUSTOM_NETWORK_NAME=docker-network
    export TARGET_HOST=10.10.1.5
    export LOGSTASH_HOST=10.10.1.5
    export INITIAL_ADMIN_USER=juansmith
    export INITIAL_ADMIN_PASSWORD_PLAIN='n0ty0urUserPls!'
    export ADOP_CLI_USER=$INITIAL_ADMIN_USER
    export ADOP_CLI_PASSWORD=$INITIAL_ADMIN_PASSWORD_PLAIN
    EOF
    ```
    
- Set the LDAP credentials and load environment variables.

    ```bash
    # Load adop default variables
    source env.config.sh
    # Load your preference variables
    source my-vars.sh
    # Generate a `platform.secrets.sh` file that will automatically be sourced.
    source credentials.generate.sh
    ```

- Deploy the containers

    ```bash
    # Create the docker network
    docker network create ${CUSTOM_NETWORK_NAME}
    
    # Create the mail-server configurations
    compose/mail-server/setup_mailserver.sh
    # Deploy mail-server container
    docker-compose -f compose/mail-server/docker-compose.yml up -d
    
    # Deploy ELK stack containers
    docker-compose -f compose/elk.yml up -d
    
    ## Deploy PDC extensions of ADOP
    # Set docker-compose overrides
    COMPOSE_OVERRIDES='f docker-compose.yml -f compose/gitlab/docker-compose.yml -f compose/jenkins-aws-vars/docker-compose.yml -f compose/jenkins-ansible-slave/docker-compose.yml'
    # Deploy
    docker-compose ${COMPOSE_OVERRIDES} ${VOLUME_OVERRIDES} up -d
    ```
- Load the Gitlab Platform in Jenkins and create initial Workspace and Project.

    ```bash
    ./gitlab-load-platform
    ./adop target set -t http://${TARGET_HOST} -u ${ADOP_CLI_USER} -p ${ADOP_CLI_PASSWORD}
    source ./.adop/target
    ./adop workspace -w ${ADOP_WORKSPACE} create -a ${INITIAL_ADMIN_USER}@${LDAP_DOMAIN}
    ./adop project -w ${ADOP_WORKSPACE} -p ${ADOP_WORKSPACE}_Project create -a ${INITIAL_ADMIN_USER}@${LDAP_DOMAIN}
    ```
