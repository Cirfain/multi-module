pipeline {
   agent none 

    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '10')
        timeout(time: 2, unit: 'HOURS')
    }

    stages 
    {
        // stage('Build and tests')
        // {
        //     agent any
        //     steps  {
        //         echo 'Unit test et packaging'
        //         //sh "mvn -Dmaven.test.failure.ignore=true clean package"
        //         //tarGz sourceDir:'.', extensions:['xml','java'], outputDir:'Archives'
        //         // script{
        //         //     node{
        //         //         docker.image('openjdk:17-alpine').inside {

        //         //         git 'branch: 'dev', url: '/home/plb/mywork/multi-module',credentialsId: 'scplb''
        //         //         sh './mvnw -B clean install'}
        //         //     } 
        //         // }

        //     }
        //         post 
        //         {
        //             always
        //             {
        //                 junit '**/target/surefire-reports/*.xml'
        //             }
        //             success
        //             {
        //                 // One or more steps need to be included within each condition's block.
        //                 echo 'Succes'
        //                 // archiveArtifacts artifacts: 'application/**/*.jar', followSymlinks: false
        //                 // stash includes: 'application/**/*.jar', name: 'file'
        //             }
        //             failure
        //             {
        //                 mail bcc: '', body: 'Ton Jenkins plante bouffon !', cc: '', from: '', replyTo: '', subject: 'Plantage', to: 'olivier.chossade@free.fr'
        //             }  
        //         } 

        // }
        stage('Kube') {
            agent {
                kubernetes {
                    inheritFrom 'jdk17-agent'
                } 
            }  
            steps {
                //unstash 'file'
                container(name:'openjdk-17') { 
                sh './mvnw -Dmaven.test.failure.ignore=true clean package'
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
                        //sh 'mvn -DskipTests verify'
                    }
                }
                 stage('Analyse Sonar') {
                    agent any
                     steps {
                        echo 'Analyse sonar'
                        sh 'mvn -Dsonar.token=${SONAR_TOKEN} clean integration-test sonar:sonar'
                    //     script
                    //    {
                    //     checkSonarQualityGate()
                    //    } 
                    }
                    
                }
            }
            
        }
            
        stage('Déploiement intégration') {
            agent none
            input {
            message 'Voulez-vous déployer ?'
                parameters {
                    booleanParam 'Deploi'
                }
            }
           
            steps 
            {
                script 
                {
                    if (Deploi)
                    { 
                        node{
                            git(branch: 'dev', url: '/home/plb/mywork/multi-module');
                            def json = readJSON(file: 'deployment.json', text: '');
                            def lstDC = json["dataCenters"];
                            println("Déploiement intégration");
                            unstash('file');
                            for (def dc in lstDC)
                            {
                                sh "cp application/**/*.jar /home/plb/mywork/environments/${dc}.jar";
                            }  
                        } 
                    } 
               } 
            }
        }

    }
    
}
def checkSonarQualityGate(){
    // Get properties from report file to call SonarQube 
    def sonarReportProps = readProperties  file: 'target/sonar/report-task.txt'
    def sonarServerUrl = sonarReportProps['serverUrl']
    def ceTaskUrl = sonarReportProps['ceTaskUrl']
    def ceTask

    // Get task informations to get the status
    timeout(time: 4, unit: 'MINUTES') {
        waitUntil(initialRecurrencePeriod: 1000)  {
            withCredentials ([string(credentialsId: 'ad53038b-7bd5-41ef-9056-d84df2962bdb', variable : 'token')]) {
                def response = sh(script: "curl -u ${token}: ${ceTaskUrl}", returnStdout: true).trim()
                ceTask = readJSON text: response
            }

            echo ceTask.toString()
              return "SUCCESS".equals(ceTask['task']['status'])
        }
    }

    // Get project analysis informations to check the status
    def ceTaskAnalysisId = ceTask['task']['analysisId']
    def qualitygate

    withCredentials ([string(credentialsId: 'ad53038b-7bd5-41ef-9056-d84df2962bdb', variable : 'token')]) {
        def response = sh(script: "curl -u ${token}: ${sonarServerUrl}/api/qualitygates/project_status?analysisId=${ceTaskAnalysisId}", returnStdout: true).trim()
        qualitygate =  readJSON text: response
    }

    echo qualitygate.toString()
    if ("ERROR".equals(qualitygate['projectStatus']['status'])) {
        error "Quality Gate failure"
    }
}
