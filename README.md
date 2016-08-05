## Launching

- Run: export TARGET\_HOST=\<IP\_OF\_PUBLIC\_HOST\>
- Run: export CUSTOM\_NETWORK\_NAME=\<CUSTOM\_NETWORK\_NAME\>
- Create a custom network: docker network create $CUSTOM\_NETWORK\_NAME
- Run: docker-compose -f compose/elk.yml up -d
- Run: export LOGSTASH\_HOST=\<IP\_OF\_LOGSTASH\_HOST\>
- Source all the required parameters for your chosen cloud provider.
    - For example, for AWS you will need to source AWS_VPC_ID, AWS_SUBNET_ID, AWS_KEYPAIR, AWS_INSTANCE_TYPE and AWS_DEFAULT_REGION. To do this, make a copy of the env.aws.provider.sh.example file in /conf/provider/examples and save it as env.provider.aws.sh in /conf/provider. You can then replace all the tokens with your values.
    - You should then run: source ./conf/env.provider.sh (this will source all the provider-specific environment variable files you have specified).
    - The provider-specific environment variable files should not be uploaded to a remote repository, hence they should not be removed from the .gitignore file.
- Run: source credentials.generate.sh \[This creates a file containing your generated passwords, platform.secrets.sh, which is sourced. If the file already exists, it will not be created.\]
    - platform.secrets.sh should not be uploaded to a remote repository hence **do not remove this file from the .gitignore file**
- Run: source env.config.sh
    - **If you delete platform.secrets.sh or if you edit any of the variables manually, you will need to re-run credentials.generate.sh in order to recreate the file or re-source the variables.**
    - **If you change the values in platform.secrets.sh, you will need to remove your existing docker containers and re-run docker-compose in order to re-create the containers with the new password values.**
    - **When creating a new instance of ADOP, you must delete platform.secrets.sh and regenerate it using credentials.generate.sh, else your old environment variables will get sourced as opposed to the new ones.**
- Choose a volume driver - either "local" or "nfs" are provided, and if the latter is chosen then an NFS server is expected along with the NFS\_HOST environment variable
- Pull the images first (this is because we can't set dependencies in Compose yet so we want everything to start at the same time): docker-compose pull
- Run (logging driver file optional): docker-compose -f docker-compose.yml -f etc/volumes/\<VOLUME_DRIVER\>/default.yml -f etc/logging/syslog/default.yml up -d

# Required environment variable on the host

- MACHINE\_NAME the name of your docker machine
- TARGET\_HOST the dns/ip of proxy
- LOGSTASH\_HOST the dns/ip of logstash
- CUSTOM\_NETWORK\_NAME: The name of the pre-created custom network to use
- [OPTIONAL] NFS\_HOST: The DNS/IP of your NFS server

# Using the platform 

###### Generate ssl certificate

Create ssl certificate for jenkins to allow connectivity with docker engine.

* RUN : source ./conf/env.provider.sh
* RUN : source credentials.generate.sh
* RUN : source env.config.sh
* RUN : ./adop compose gen-certs ${DOCKER\_CLIENT\_CERT\_PATH}

Note : For Windows run this command from a terminal (Git Bash) as administrator.

###### Load Platform

* Access the target host url `http://<TARGET_HOST>` with the your username and password
 * This page presents the links to all the tools.
* Click: Jenkins link.
* Run: Load\_Platform job
 * Once the Load\_Platform job and other downstream jobs are finished your platform is ready to be used.
 * This job generates a example workspace folder, example project folder and jenkins jobs/pipelines for java reference application.
* Create environment to deploy the reference application
 * Navigate to `http://<TARGET_HOST>/jenkins/job/ExampleWorkspace/job/ExampleProject/job/Create_Environment`
 * Build with Parameters keeping the default value.
* Run Example pipeline
 * Navigate to `http://<TARGET_HOST>/jenkins/job/ExampleWorkspace/job/ExampleProject/view/Java_Reference_Application/`
 * Click on run.
* Browse the environment
 * Click on the url for your environment from deploy job.
 * You should be able to see the spring petclinic application.
* Now, you can clone the repository from gerrit and make a code change to see the example pipeline triggered automatically.

# Define Default Elastic Search Index Pattern

Kibana 4 does not provide a configuration property that allow to define the default index pattern so the following manual procedure should be adopted in order to define an index pattern:

- Navidate to Settings > Indices using Kibana dashboard
- Set index name or pattern as "logstash-*"
- For the below drop-down select @timestamp for the Time-field name
- Click on create button


