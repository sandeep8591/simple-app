pipeline {
    agent any
    tools {
        maven 'maven'
        jdk 'jdk 8'
    }
    options {
        buildDiscarder logRotator(daysToKeepStr: '5', numToKeepStr: '7')
    }
    def gitBranch = env.BRANCH_NAME
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
                             file: "target/${mavenPom.name}-${mavenPom.version}.war",
                             type: 'war'
                              ]
                            ],
                      credentialsId: 'nexus-cred',
                      groupId: 'in.javahome',
                      nexusUrl: '192.168.1.164:8081',
                      nexusVersion: 'nexus3',
                      protocol: 'http',
                      repository: 'repository-example',
                      version: "${mavenPom.version}"
                    }
            }
        }
   }
}
