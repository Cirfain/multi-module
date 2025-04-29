pipeline {
   agent any 
    tools {
      maven 'Maven 3'
        jdk 'Java21'
        }



    stages {
        stage('Build and tests') {
            
            steps {
                echo 'Unit test et packaging'
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                        }
            } 
             
        }
        stage('Analyse qualité et vulnérabilités') {
            parallel {
                stage('Vulnérabilités') {
                    steps {
                        echo 'Tests de Vulnérabilités OWASP'
                    }
                    
                }
                 stage('Analyse Sonar') {
                     steps {
                        echo 'Analyse sonar'
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

