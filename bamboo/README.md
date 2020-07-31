# spring-boot-hello

Pre-requisites:
-----
  - Install Java
  - Install GIT
  - Install Maven
  - Install Docker
  - Install Jenkins
  - Create ECR Repo with the name of "hellospringboot"
  - Create a policy for ECS full Access with name of "AmazonECSFullAccess"
  - Create IAM Role with the name of "ecrFullAccess-ecsTaskExecutionRole" also add below policies
      * AmazonEC2ContainerRegistryFullAccess
      * AmazonECSTaskExecutionRolePolicy
      * AmazonECSFullAccess
  
Create Maven Job:
-------
1)  SCM: 
--------
    
    https://github.com/VamsiTechTuts/spring-boot-hello.git
2)  Maven:
----------

    Give name for Maven and also give "clean install"

3) Push image to ECR:
---------------------

    #!/usr/bin/env bash
    export AWS_DEFAULT_REGION=us-west-2
    $(aws ecr get-login --no-include-email --region us-west-2)
    DOCKER_REPO=`aws ecr describe-repositories --repository-names hellospringboot | grep repositoryUri | cut -d "\"" -f 4`
    docker build --no-cache -t ${DOCKER_REPO}:1.0 .
    docker push ${DOCKER_REPO}:1.0

4)  Deploy Spring boot application On ECS:
------------------------------------------

    #!/usr/bin/env bash
    export AWS_DEFAULT_REGION=us-west-2
    dockerRepo=`aws ecr describe-repositories --repository-name hellospringboot --region us-west-2 | grep repositoryUri | cut -d "\"" -f 4`
    dockerTag=`aws ecr list-images --repository-name hellospringboot --region us-west-2 | grep imageTag | head -n 1 | cut -d "\"" -f 4`
    sed -e "s;DOCKER_IMAGE_NAME;${dockerRepo}:${dockerTag};g" ${WORKSPACE}/template.json > taskDefinition.json
    aws ecs create-cluster --cluster-name test-cluster
    aws ecs register-task-definition --family jenkins-test --cli-input-json file://taskDefinition.json --region us-west-2
    revision=`aws ecs describe-task-definition --task-definition jenkins-test --region us-west-2 | grep "revision" | tr -s " " | cut -d " " -f 3`
    aws ecs create-service --cluster test-cluster --service-name test-service --task-definition jenkins-test:${revision} --desired-count 1 --launch-type FARGATE --platform-version LATEST --network-configuration "awsvpcConfiguration={subnets=[subnet-8a1fdcf2],securityGroups=[sg-f9abbfaa],assignPublicIp=ENABLED}"
    
  

