pipeline{
    //Directives
    agent any
    tools {
        maven 'maven'
    }
    environment{
       ArtifactId = readMavenPom().getArtifactId()
       Version = readMavenPom().getVersion()
       Name = readMavenPom().getName()
       GroupId = readMavenPom().getGroupId()
    }


    stages {
        // Specify various stage with in stages


       // stage 1.0 Secrets Scan
        stage ('Secrets Scan'){
            steps {
                echo " Scanning github repo for secrets ...."
                sshPublisher(publishers:
                [sshPublisherDesc(
                    configName: 'sectools_server',
                    transfers: [
                        sshTransfer(
                                cleanRemote: false,
                                //execCommand: 'docker run --rm -v "/home/ansibleadmin:/pwd" trufflesecurity/trufflehog:latest --fail --json-legacy --no-verification github --repo http://github.com/r6m0l4/MyLab.git > /home/ansibleadmin/hog_results_${BUILD_NUMBER}.json',
                                execCommand: 'docker run --rm -v "/home/ansibleadmin:/pwd" trufflesecurity/trufflehog:latest --fail --json-legacy --no-verification github --repo ${REPO_URL} > /home/ansibleadmin/hog_results_${BUILD_NUMBER}.json',
                                execTimeout: 300000
                        )
                    ],
                    //continueOnError : true,
                    //failOnError : true,
                    usePromotionTimestamp: false,
                    useWorkspaceInPromotion: false,
                    verbose: true)
                    ])
                
            }

            post{
                success {
                }
                failure {
                    script{
                        sh "exit 1"
                        //or
                        error "Failed, exiting now..."
                    }
                }
                unstable {
                    script{
                           sh "exit 1"
                          //or
                          error "Unstable, exiting now..."                    
                     }
                }
            }


        }






        // stage 2 Build
        stage ('Build'){
            steps {
                sh 'mvn clean install package'
            }
        }

     
        


        // Stage3 : Publish the artifacts to Nexus
        stage ('Publish to Nexus'){
            steps {

             script {

                def NexusRepo = Version.endsWith("SNAPSHOT") ? "MyLab-SNAPSHOT" : "MyLab-RELEASE"

                nexusArtifactUploader artifacts: 
                [[artifactId: "${ArtifactId}" , 
                classifier: '', 
                file: "target/${ArtifactId}-${Version}.war", 
                type: 'war']], 
                credentialsId: '3b015235-520c-4cf7-9ebe-42569b501604',
                groupId: "${GroupId}", 
                nexusUrl: '172.20.10.141:8081',
                nexusVersion: 'nexus3', 
                protocol: 'http', 
                repository: "${NexusRepo}", 
                version: "${Version}"

               }        
            }
        }




        // Stage3.1 : Publish the source code to Sonarqube
        stage ('Sonarqube SAST Analysis'){
            steps {
                echo ' Source code published to Sonarqube for SCA......'
                withSonarQubeEnv('sonarqube'){ // You can override the credential to be used
                     sh 'mvn sonar:sonar'
                }

            }
        }




        // Stage 4 : Print some information
        stage ('Print Environment variables'){
                    steps {
                        echo "Artifact ID is '${ArtifactId}'"
                        echo "Version is '${Version}'"
                        echo "GroupID is '${GroupId}'"
                        echo "Name is '${Name}'"
                    }
                }



       // Stage5 : Deploying to Tomcat
        stage ('Deploy to Tomcat'){
            steps {
                echo "Deploying to tomcat ...."
                sshPublisher(publishers: 
                [sshPublisherDesc(
                    configName: 'ansible_controller', 
                    transfers: [
                        sshTransfer(
                                cleanRemote: false,
                                execCommand: 'ansible-playbook /opt/playbooks/downloadanddeploy_as_tomcat_user.yml -i /opt/playbooks/hosts',
                                execTimeout: 120000
                        )
                    ], 
                    usePromotionTimestamp: false, 
                    useWorkspaceInPromotion: false, 
                    verbose: false)
                    ])
            
            }
        }




      // stage 5.5 ZAP DAST Baseline Scan
        stage ('ZAP DAST Baseline Scan'){
            steps {
                echo " Performing ZAP DAST baseline scan ...."
                sshPublisher(publishers:
                [sshPublisherDesc(
                    configName: 'sectools_server',
                    transfers: [
                        sshTransfer(
                                cleanRemote: false,
                                execCommand: 'docker run --rm -v "$(pwd):/zap/wrk/:rw" -p 8080:8080  --user root -i owasp/zap2docker-stable zap-baseline.py -t ${TOMCAT_TEST_URL} -g gen.conf -r /zap/wrk/zap_baseline_report_${BUILD_NUMBER}.html',
                                execTimeout: 500000
                        )
                    ],
                    usePromotionTimestamp: false,
                    useWorkspaceInPromotion: false,
                    verbose: true)
                    ])
                
            }
        }




       // Stage 6 : Deploying the build artifact to Docker
        stage ('Deploy to Docker'){
            steps {
                echo "Deploying to docker ...."
                sshPublisher(publishers: 
                [sshPublisherDesc(
                    configName: 'ansible_controller', 
                    transfers: [
                        sshTransfer(
                                cleanRemote:false,
                                execCommand: 'ansible-playbook /opt/playbooks/downloadanddeploy_docker.yml -i /opt/playbooks/hosts',
                                execTimeout: 120000
                        )
                    ], 
                    usePromotionTimestamp: false, 
                    useWorkspaceInPromotion: false, 
                    verbose: false)
                    ])
            
            }
        }








/*

      // stage 7 ZAP DAST Full Scan
        stage ('ZAP DAST Full Scan'){
            steps {
                echo " Performing ZAP DAST full scan ...."
                sshPublisher(publishers:
                [sshPublisherDesc(
                    configName: 'sectools_server',
                    transfers: [
                        sshTransfer(
                                cleanRemote: false,
                                execCommand: 'docker run --rm -v "$(pwd):/zap/wrk/:rw" -p 8080:8080  --user root -i owasp/zap2docker-stable zap-full-scan.py -t ${TOMCAT_TEST_URL} -g gen.conf -r /zap/wrk/zap_full_report_${BUILD_NUMBER}.html',
                                execTimeout: 1000000
                        )
                    ],
                    usePromotionTimestamp: false,
                    useWorkspaceInPromotion: false,
                    verbose: true)
                    ])
                
            }
        }

*/






         
    }    
         
}   


