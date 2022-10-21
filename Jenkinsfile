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

        // stage 1. Build
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

        
        
        // Stage3 : Publish Artifacts to Nexus       
//        stage ('Publish to Nexus'){
//            steps {
//                nexusArtifactUploader artifacts: [[artifactId: 'VinayDevOpsLab', classifier: '', file: 'target/VinayDevOpsLab-0.0.1-SNAPSHOT.war', type: 'war']], credentialsId:'3b015235-520c-4cf7-9ebe-42569b501604', groupId: 
//'com.vinaysdevopslab', nexusUrl: '172.20.10.141:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'MyLab-SNAPSHOT', version:    
//'0.0.1-SNAPSHOT'
//            }
//        }





        // Stage3 : Publish the artifacts to Nexus
        stage ('Publish to Nexus'){
            steps {

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
                repository: 'MyLab-SNAPSHOT', 
                version: "${Version}"
        
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
        stage ('Deploy to tomcat'){
            steps {
                echo ' deploying to Toomcat...'
    
            }
        }
        


      // Stage 6 : Deploying the build artifact to Docker
        stage ('Deploy to Docker'){
            steps {    
                echo "Deploying to Docker ...."
            }
        }



         
    }    
         
}   


