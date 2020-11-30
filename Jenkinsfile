def gitBranch = env.GIT_BRANCH
pipeline {
    environment {
        registry = "vennamsandeep/testjava" 
        registryCredential = 'dockerhub_id' 
        dockerImage = '' 
    }
    agent any
    tools {
        maven 'maven'
        jdk 'jdk 8'
    }
    options {
        buildDiscarder logRotator(daysToKeepStr: '5', numToKeepStr: '7')
    }
    stages{
        stage('Build'){
            steps{
                 sh """
                 mvn clean package
                 mvn sonar:sonar
                 """
                 archiveArtifacts artifacts: 'target/*.war', onlyIfSuccessful: true
            }
        }
        {
         stage('Jfrog Artifactory')
            steps {
                script {
                      def server = Artifactory.server 'artifactory'
                      def rtMaven = Artifactory.newMavenBuild()
                      rtMaven.deployer server: server, releaseRepo: 'jfrog-local', snapshotRepo: 'jfrog-local'
                      def buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install'
                    }
            }
        }
        stage('Docker build') {
           steps {
           sh 'docker build -t vennamsandeep/testjava:${BUILD_NUMBER} .' 
          }
        }
        stage('Docker registry') { 
            steps { 
                script {
                    docker.withRegistry( '', registryCredential ) { 
                    sh 'docker push vennamsandeep/testjava:${BUILD_NUMBER}'
                    }
                }
            }
        }
       stage('Run container ') {
        steps {
            sh 'docker run -itd -p 80:8080 vennamsandeep/testjava:${BUILD_NUMBER}'
            }    
       }
   }
}
