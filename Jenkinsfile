pipeline {
    agent none

    stages {
    //Execution with First Container   
      stage('Build') {
        agent {
            kubernetes {
            //cloud 'kubernetes'
                containerTemplate {
                name 'mvnbuild'
                image 'maven:3-alpine'
                ttyEnabled true
                command 'cat'
                }
            }
        }
        stages {
            stage('Build War'){
                steps {
                    sh 'mvn -B -DskipTests clean package'
                    sh 'ls target'
                    }
            }
            stage('Upload War'){
                steps {
                    nexusArtifactUploader artifacts: [
                        [
                            artifactId: 'mcms-webapp', 
                            classifier: '', 
                            file: 'target/mcms-webapp-0.0.1.war', 
                            type: 'war'
                        ]
                    ], 
                    credentialsId: 'nexus_id', 
                    groupId: 'com.mcms.devsecops', 
                    nexusUrl: '10.95.8.137:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: 'maven-releases', 
                    version: '0.0.1'
                sh "sleep 10"
                }
            }

        } // Build Stage Completes
        
      }
    //Execution with Second Container 
       stage('Dockerize') {
         agent {
             label 'gitops'
         }
            // kubernetes {
            // //cloud 'kubernetes'
            //     containerTemplate {
            //     name 'dockercli'
            //     image 'gustavoapolinario/jenkins-docker'
            //     ttyEnabled true
            //     command 'cat'
            //     }
            // }
        

        stages{
            stage('PublishtoRegistry'){
                steps{

                    sh "wget http://10.95.8.137:8081/repository/maven-releases/com/mcms/devsecops/mcms-webapp/0.0.1/mcms-webapp-0.0.1.war"
                    sh "cp mcms-webapp-0.0.1.war mcms-webapp.war"
                    sh "rm -f mcms-webapp-0.0.1.war"
                    sh "ls *war*"
                    sh 'docker build -f Dockerfile -t mcmsdemo/webapp .'
                    withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        //sh "docker login quay.io -u ${USERNAME} -p ${PASSWORD}"
                        sh "docker login http://10.95.8.137:1111/repository/mcms-hub/ -u ${USERNAME} -p ${PASSWORD}"
                        sh "docker tag mcmsdemo/webapp 10.95.8.137:1111/repository/mcms-hub/webapp"
                        sh "docker push 10.95.8.137:1111/repository/mcms-hub/webapp"
                        sh "docker rmi mcmsdemo/webapp 10.95.8.137:1111/repository/mcms-hub/webapp"
                        sh "export KUBECONFIG=/root/.kube/config-iscpconfig"
                        sh "oc get nodes"
                        sh "oc apply -f deploy_to_ocp"
                    }
                }

            }
        }
      }

   }     
}


    //   stage('Create Docker Image') {
    //     steps {
    //         sh 'docker build -f Dockerfile -t mcmsdemo/webapp .'
    //         sh 'docker images'
    //     }
    //   }