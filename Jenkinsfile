pipeline{
    //Directives
    agent any
    tools {
        maven 'maven'
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
        stage ('Publish to Nexus'){
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'VinayDevOpsLab', classifier: '', file: 'target/VinayDevOpsLab-0.0.1-SNAPSHOT.war', type: 'war']], credentialsId:'3b015235-520c-4cf7-9ebe-42569b501604', groupId: 'com.vinaysdevopslab', nexusUrl: '172.20.10.141:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'MyLab-SNAPSHOT', version:    
'0.0.1-SNAPSHOT'
            }
        }


        // Stage4 : Deploying
        stage ('Deploying'){ 
            steps {
                echo ' deploying...'
                
            }   
        }
         
    }    
         
}   


