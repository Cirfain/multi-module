pipeline {
   agent any 
    

    stages {
        stage('Build and tests') {
            
            steps {
                echo 'Unit test et packaging'
                tool name : "Maven 3"
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
            post {
                always {
                    junit 'pom.xml'
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

