pipeline{
    //Directives
    agent any // it executes pipiline on any available agent and creates workspace for that pipe
    tools {
        maven 'maven'
    }

    // use pipeline utility step to read pom.xml file
    environment {
        GroupId = readMavenPom().getGroupId()
        ArtifactId = readMavenPom().getArtifactId()
        Version = readMavenPom().getVersion()
        Name = readMavenPom().getName()
    }

    stages {
        // Specify various stage with in stages

        // stage 1. Build
        stage ('Build'){
            steps {
                sh 'mvn clean install package' // execute shell delete all previously compiled Java
            }
        }

   /*      // Stage2 : Testing
        stage ('Test'){
            steps {
                echo ' testing......'

            }
        } */

    // stage3 : publish to Nexus
      stage ('Publish to Nexus'){
            steps {
                script { // If V is SNAPSHOT publish it on SNAPSHOT repo on Nexus, else on RELEASE
                    def NexusRepo = Version.endsWith("SNAPSHOT") ?  "DevOps-SNAPSHOT" : "DevOps-RELEASE" 
                    nexusArtifactUploader artifacts:
                    [[artifactId: "${ArtifactId}",
                    classifier: '',
                    file: "target/${ArtifactId}-${Version}.war",
                    type: 'war']],
                    credentialsId: 'e7d581e4-63c6-4f19-a89c-1aea6f6a49d5',
                    groupId: "${GroupId}", 
                    nexusUrl: '172.20.0.199:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository:"${NexusRepo}", 
                    version: "${Version}"  
                 }                
                     }

      }


  /*     // stage 4 : print some infos
        stage ('Print Environment variables'){
            steps {
                echo "GroupID is '{$GroupId}'"
                echo "Artifect ID is '{$ArtifactId}'"
                echo "Version  is '{$Version}'"              
                echo "Name is '{$Name}'"
            }
        } */

          // Stage 5 : Deploying the build artifact to Apache Tomcat
        stage ('Deploy to Tomcat'){
            steps {
                echo "Deploying ...."
                sshPublisher(publishers: 
                [sshPublisherDesc(
                    configName: 'Ansible_Controller', 
                    transfers: [
                        sshTransfer(
                                cleanRemote:false,
                                execCommand: 'ansible-playbook /opt/playbooks/downloadanddeploy_as_tomcat_user.yaml -i /opt/playbooks/hosts',
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
                echo "Deploying ...."
                sshPublisher(publishers: 
                [sshPublisherDesc(
                    configName: 'Ansible_Controller', 
                    transfers: [
                        sshTransfer(
                                cleanRemote:false, 
                                execCommand: 'ansible-playbook /opt/playbooks/downloadanddeploy_docker.yaml -i /opt/playbooks/hosts',
                                execTimeout: 120000
                        )
                    ], 
                    usePromotionTimestamp: false, 
                    useWorkspaceInPromotion: false, 
                    verbose: false)
                    ])
            
            }
        }   */
         

     // Stage7 : Publish the source code to Sonarqube
        stage ('Sonarqube Analysis'){
           
            steps {
                echo ' Source code published to Sonarqube for SCA......'
                withSonarQubeEnv('sonarqube'){ // You can override the credential to be used
                     sh 'mvn sonar:sonar'
                    
                }

            }
        } 

        
        
    }
}
