pipeline {
  environment {
    PROJECT     = 'mongodb-springboot'
    ECR_REGISTRY = "350100602815.dkr.ecr.eu-west-2.amazonaws.com/mongodb-springboot"
    AWS_REGION = "eu-west-2"
    DOCKERFILE = "Dockerfile"
    //MAVEN_OPTS = "-Dmaven.repo.local=$WORKSPACE/.m2"
    COMPOSE_FILE = "docker-compose.yml"
    EC2_INSTANCE = "ec2-user@ec2-18-132-193-87.eu-west-2.compute.amazonaws.com"
  }
  
  agent any
  
  stages {
    
    stage("Initial cleanup") {
        steps {
        dir("${WORKSPACE}") {
            deleteDir()
        }
        }
    }
    
    
    stage('Checkout') {
      steps {
      checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'e1868d62-3cd4-44da-aba1-a24e2183d6e3', url: 'https://github.com/earchibong/springboot_project.git']])
      }
    }
    
    stage('Build preparations') {
      steps {
            script {
                  // calculate GIT lastest commit short-hash
                  gitCommitHash = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                  shortCommitHash = gitCommitHash.take(7)
                  // calculate a sample version tag
                  VERSION = shortCommitHash
                  // set the build display name
                  currentBuild.displayName = "#${BUILD_ID}-${VERSION}"
                  IMAGE_NAME = "$PROJECT:$VERSION"
            }
      }
    }   
    
    //stage('Build Jar file') {
      //steps {
        //sh "mvn -f ${env.WORKSPACE}/pom.xml clean package -DskipTests"
        //archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      //}
    //}
    
    stage('Build Docker image') {
      steps {
        script {
               sh """aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}"""
               sh "docker build --tag $IMAGE_NAME --file ${DOCKERFILE} ${env.WORKSPACE}"
                docker.withRegistry("https://${ECR_REGISTRY}") {
                docker.image("$IMAGE_NAME").push("$prod-BUILD_NUMBER")
              }
              
        }
      }
    }

    stage('Deploy to EC2') {
      steps {
        script {
              sh "docker pull ${ECR_REGISTRY}:${prod-BUILD_NUMBER}"
              sh """scp  -o StrictHostKeyChecking=no ${COMPOSE_FILE} ${EC2_INSTANCE}:~/docker-compose.yml"""
              sh """ssh  -o StrictHostKeyChecking=no ${EC2_INSTANCE} 'docker-compose -f ~/docker-compose.yml up -d'"""
        }
      }
      
    }
  }

          post
    {
        always
        {
            sh "docker rmi -f $IMAGE_NAME "
        }
    }
}