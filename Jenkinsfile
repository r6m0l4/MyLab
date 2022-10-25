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
                echo "Scanning github repo for secrets ...."
                sshPublisher(publishers:
                [sshPublisherDesc(
                    configName: 'sectools_server',
                    transfers: [
                        sshTransfer(
                                cleanRemote: false,
                                execCommand: 'docker run --rm -v "/home/ansibleadmin:/pwd" trufflesecurity/trufflehog:latest --json-legacy --no-verification github --repo http://github.com/r6m0l4/MyLab.git > /home/ansibleadmin/hog_results_%BUILD_NUMBER%.json',
                                //execCommand: 'uname -a > /home/ansibleadmin/uname.txt',
                                execTimeout: 300000
                        )
                    ],
                    usePromotionTimestamp: false,
                    useWorkspaceInPromotion: false,
                    verbose: true)
                    ])
                
            }
        }






        // stage 1.5 Build
        stage ('Build'){
            steps {
                sh 'mvn clean install package'
            }
        }

        // Stage2 : Testing
        stage ('Test'){
            steps {
                echo ' testing......'

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
        stage ('Sonarqube Analysis'){
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




         
    }    
         
}   


