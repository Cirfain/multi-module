pipeline {
   agent any 
    tools 
    {
        maven 'Maven 3'
        jdk 'Java21'
    }
    environment {
        SONAR_TOKEN = credentials('ad53038b-7bd5-41ef-9056-d84df2962bdb')
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
                        stash includes: 'application/**/*.jar', name: 'file'
                    }
                    failure
                    {
                        mail bcc: '', body: 'Ton Jenkins plante bouffon !', cc: '', from: '', replyTo: '', subject: 'Plantage', to: 'olivier.chossade@free.fr'
                    }  
                } 

        }

        stage('Analyse qualité et vulnérabilités')
        {
            parallel
            {
                stage('Vulnérabilités') {
                    agent any
                    steps {
                        echo 'Tests de Vulnérabilités OWASP'
                        sh 'mvn -DskipTests verify'
                    }
                }
                 stage('Analyse Sonar') {
                    agent any
                     steps {
                        echo 'Analyse sonar'
                        sh 'mvn -Dsonar.token=${SONAR_TOKEN} clean integration-test sonar:sonar'
                     }
                    
                }
            }
            
        }
            
        stage('Déploiement intégration') {
            input {
              message 'Dans quel datacenter voulez-vous deployer votre truc ?'
                parameters {
                    choice choices: ['Paris', 'Lille', 'Lyon'], name: 'VILLES'
                }
            }
            steps {
                echo "Déploiement intégration"
                unstash 'file'
                sh 'cp application/**/*.jar ${VILLES}.jar'
            }
        }

    }
    
}

