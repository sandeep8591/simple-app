def gitBranch = env.GIT_BRANCH
pipeline {
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
        stage('Upload War To Nexus'){
            steps{
                script{
                      def mavenPom = readMavenPom file: 'pom.xml'
                      nexusArtifactUploader artifacts: [
                            [artifactId: 'simple-app',
                             classifier: '',
                             file: "target/simple-app-${mavenPom.version}.war",
                             type: 'war'
                              ]
                            ],
                      credentialsId: 'nexus-cred',
                      groupId: 'in.javahome',
                      nexusUrl: '192.168.0.164:8081',
                      nexusVersion: 'nexus3',
                      protocol: 'http',
                      repository: 'repository-example',
                      version: "${mavenPom.version}"
                    }
            }
        }
        stage('Build dockerimage') {
            steps {
                sh 'docker build -it vennamsandeep/testjava:${BUILD_NUMBER} .'
                sh 'docker run -itd -p 80:8080 vennamsandeep/testjava:${BUILD_NUMBER}'
            }
        }
   }
}
