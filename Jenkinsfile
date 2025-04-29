pipeline {
   agent any 
    tools 
    {
        maven 'Maven 3'
        jdk 'Java21'
    }
    environment {
        SONAR_TOKEN = credentials('Token Sonaqube')
    }
    stages 
    {
        stage('Build and tests')
        {
            steps  {
                echo 'Unit test et packaging'
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
                            post 
                {
                    always
                    {
                        junit '**/target/surefire-reports/*.xml'
                    }
                    success
                    {
                        // One or more steps need to be included within each condition's block.
                        archiveArtifacts artifacts: 'application/**/*.jar', followSymlinks: false
                    }
                    failure
                    {
                        mail bcc: '', body: 'Ton Jenkins plante bouffon !', cc: '', from: '', replyTo: '', subject: 'Plantage', to: 'olivier.chossade@free.fr'
                    }  
                } 

        }

        stage('Analyse qualité et vulnérabilités')
        {

// mvn -DskipTests verify

// Une tâche exécutant une analyse SonarQube
// mvn -Dsonar.token=${SONAR_TOKEN} clean integration-test sonar:sonar

            parallel
            {
                stage('Vulnérabilités') {
                    steps {
                        echo 'Tests de Vulnérabilités OWASP'
                        sh "mvn -DskipTests verify"
                    }
                }
                 stage('Analyse Sonar') {
                     steps {
                        echo 'Analyse sonar'
                        sh "mvn -Dsonar.token=${SONAR_TOKEN} clean integration-test sonar:sonar"
                     }
                    
                }
            }
            
        }
            
        stage('Déploiement intégration') {

            steps {
                echo "Déploiement intégration"
                
            }
        }

    }
    
}

