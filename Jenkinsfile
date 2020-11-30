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
#        stage('Upload War To Nexus'){
#            steps {
#                script {
#                      def mavenPom = readMavenPom file: 'pom.xml'
#                      nexusArtifactUploader artifacts: [
#                            [artifactId: 'simple-app',
#                             classifier: '',
#                             file: "target/simple-app-${mavenPom.version}.war",
#                             type: 'war'
#                              ]
#                            ],
#                      credentialsId: 'nexus-cred',
#                      groupId: 'in.javahome',
#                      nexusUrl: '192.168.0.164:8081',
#                      nexusVersion: 'nexus3',
#                      protocol: 'http',
#                      repository: 'repository-example',
#                      version: "${mavenPom.version}"
#                    }
#            }
#        }
        {
         stage('Jfrog Artifactory')
            steps {
                script {
                      def server = Artifactory.server 'artifactory'
                      def rtMaven = Artifactory.newMavenBuild()
                      rtMaven.deployer server: server, releaseRepo: 'jfrog-local', snapshotRepo: 'jfrog-local'
                      def buildInfo = rtMaven.run pom: 'simple-app/pom.xml', goals: 'clean install'
                    }
            }
        }
        stage('Docker build') {
           steps {
           sh 'docker build -t vennamsandeep/testjava:${BUILD_NUMBER} .' 
          }
        }
        stage('Deploy our image') { 
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
