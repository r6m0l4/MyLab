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


        // Stage 0 : Print some information
        stage ('Print Environment variables'){
                    steps {
                        echo "Artifact ID is '${ArtifactId}'"
                        echo "Version is '${Version}'"
                        echo "GroupID is '${GroupId}'"
                        echo "Name is '${Name}'"
                        echo "Workspace is '${WORKSPACE}'"
                        echo "Jenkins home is '${JENKINS_HOME}'"
                        echo "Repo URL is '${REPO_URL}'"
                        echo "Job name is '${JOB_NAME}'"
                        echo "Job base name is '${JOB_BASE_NAME}'"
                        echo "Build number is '${BUILD_NUMBER}'"
                        echo "Build tag is '${BUILD_TAG}'"
                    }
                }


       // Stage 0.1 : Test SSH Agent to Sectools
        stage ('SSH Agent Test'){
                    steps {
                        echo " Testing SSH connecting to remote server ...."
                        sshagent(credentials: ['sectools_key']) {
                            sh 'ssh -o StrictHostKeyChecking=no ansibleadmin@${SECTOOLS_IP} touch /home/ansibleadmin/ssh-agent_test_${BUILD_TAG}.check'
                            }
                    }
                }




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
                                execCommand: 'docker run --rm -v "/home/ansibleadmin:/pwd" trufflesecurity/trufflehog:latest --fail --json-legacy --no-verification github --repo ${REPO_URL} > /home/ansibleadmin/hog_results_${BUILD_TAG}.json',
                                execTimeout: 300000
                        )
                    ],
                    usePromotionTimestamp: false,
                    useWorkspaceInPromotion: false,
                    verbose: true)
                    ])
                
            }

            //Determine action based on scan results
            post{
                success {
                    echo "exit 0 - No issues found, keep going"
                }
                failure {
                    script{
                        echo "scan results failed"
                        unstable('Vuls detected')
                        //currentBuild.result = 'FAILURE'
                        //sh "exit 1"
                        ////or
                        //error "Failed, exiting now..."
                    }
                }
                unstable {
                    script{
                        echo "scan results unstable"
                        unstable('Vuls detected')
                        //exit 2 == unstable
                        //currentBuild.result = 'UNSTABLE'
                        //sh "exit 2"
                        ////or
                        //error "Unstable, exiting now..."              
                     }
                }
            }


        }



       // Stage 1.1 : Copy results to Jenkins Workspace
        stage ('Retrieve Secrets Report'){
                    steps {
                        echo " Copying report from remote server to Jenkins project workspace ...."
                        sshagent(credentials: ['sectools_key']) {
                            sh 'scp -o StrictHostKeyChecking=no ansibleadmin@${SECTOOLS_IP}:/home/ansibleadmin/hog_results_${BUILD_TAG}.json ${WORKSPACE}/hog_results_${BUILD_TAG}.json'
                            }
                    }
                }







        // stage 2 Build
        stage ('Build'){
            steps {
                echo " Building application ...."
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




        // Stage4 : Publish the source code to Sonarqube
        stage ('Sonarqube SAST'){
            steps {
                echo ' Source code published to Sonarqube for SCA......'
                withSonarQubeEnv('sonarqube'){ // You can override the credential to be used
                     sh 'mvn sonar:sonar'
                }

            }
        }






       // Stage5 : Deploying to Tomcat
        stage ('Deploy to Tomcat'){
            steps {
                echo "Deploying to tomcat ...."
                sshPublisher(continueOnError: true, publishers: 
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




      // stage 6 ZAP DAST Baseline Scan
        stage ('ZAP DAST Baseline'){
            steps {
                echo " Performing ZAP DAST baseline scan ...."
                sshPublisher(continueOnError: true, publishers:
                [sshPublisherDesc(
                    configName: 'sectools_server',
                    transfers: [
                        sshTransfer(
                                cleanRemote: false,
                                execCommand: 'sudo docker run --rm -v "$(pwd):/zap/wrk/:rw" -p 8080:8080  --user root -i owasp/zap2docker-stable zap-baseline.py -t ${TOMCAT_TEST_URL} -g gen.conf -r /zap/wrk/zap_baseline_report_${BUILD_TAG}.html',
                                execTimeout: 500000
                        )
                    ],
                    usePromotionTimestamp: false,
                    useWorkspaceInPromotion: false,
                    verbose: true)
                    ])
                
            }
        }





       // Stage 7 : Copy results to Jenkins Workspace
        stage ('Retrieve ZAP DAST Report'){
                    steps {
                        echo " Copying report from remote server to Jenkins project workspace ...."
                        sleep(time:10,unit:"SECONDS")
                        sshagent(credentials: ['sectools_key']) {
                            sh 'scp -o StrictHostKeyChecking=no ansibleadmin@${SECTOOLS_IP}:/home/ansibleadmin/zap_baseline_report_${BUILD_TAG}.html ${WORKSPACE}/zap_baseline_report_${BUILD_TAG}.html'
                            }
                    }
                }




        

        // Stage 8 : Dependancy Check (only fail on CVSS == 10  --failOnCVSS "10")
        stage ('Dependency Check') {
            steps {
                echo " Performing OWASP Dependancy Check scan on workspace ...."
                dependencyCheck additionalArguments: ''' 
                    -o "./" 
                    -s "./"
                    -f "ALL" 
                    --prettyPrint''', 
                odcInstallation: 'OWASP-Dependancy-Check'
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
  

            //Determine action based on scan results
            post{
                success {
                    echo "exit 0 - No issues found, keep going"
                }
                failure {
                    script{
                        echo "scan results failed"
                        unstable('Vuls detected')
                        //currentBuild.result = 'FAILURE'
                        //sh "exit 1"
                        ////or
                        //error "Failed, exiting now..."
                    }
                }
                unstable {
                    script{
                        echo "scan results unstable"
                        unstable('Vuls detected')
                        //exit 2 == unstable
                        //currentBuild.result = 'UNSTABLE'
                        //sh "exit 2"
                        ////or
                        //error "Unstable, exiting now..."              
                     }
                }
            }


        }   







       // Stage 9 : Deploying the build artifact to Docker
        stage ('Deploy to Docker'){
            steps {
                echo "Deploying to docker ...."
                sshPublisher(continueOnError: true, publishers: 
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
        stage ('ZAP DAST Full'){
            steps {
                echo " Performing ZAP DAST full scan ...."
                sshPublisher(publishers:
                [sshPublisherDesc(
                    configName: 'sectools_server',
                    transfers: [
                        sshTransfer(
                                cleanRemote: false,
                                execCommand: 'sudo docker run --rm -v "$(pwd):/zap/wrk/:rw" -p 8080:8080  --user root -i owasp/zap2docker-stable zap-full-scan.py -t ${TOMCAT_TEST_URL} -g gen.conf -r /zap/wrk/zap_full_report_${BUILD_TAG}.html',
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


