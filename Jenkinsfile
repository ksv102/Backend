properties([pipelineTriggers([githubPush()])])
pipeline {
    agent any
    environment {
        registryUrl = "hidpdeveastusbotacr.azurecr.io"
        
        }
    
        stages {
          stage( 'Gitcheckout') {
                steps {
                 //   checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'test-tken-v', url: '']]])
                    echo "test"
                }
            }  
            stage( 'Build') {
                steps {
                    script {
                        datas = readYaml (file : "$WORKSPACE/config.yml")
                        echo "build type is: ${datas.Build_type}"
                        
                        
                        if( "${datas.Build_type}" == "maven" )
                        {
                        sh 'mvn clean install -DskipTests=True'
                        }
                        else( "${datas.Build_type}" == "gradle" ) 
                        {    
                        sh 'gradle build'
                        }
                        
                    }
                }
            }
              stage('SonarQube analysis') {
                steps {
                    withSonarQubeEnv('sonarqube-9.0.1') {
                    sh "mvn sonar:sonar -Dsonar.projectKey=maven-demo -Dsonar.host.url=http://20.55.51.162:9000 -Dsonar.login=814a6e694a4c3d536d41858ff93a80090109d2b5"
    }
        }
        }
            stage( 'Build docker image') {
                steps {
                    sh "docker build -t $registryUrl/hello:${BUILD_NUMBER} ."
                    
                }
                
            }
            stage('Upload Image to ACR') {
                steps{
                    withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'docker_pass', usernameVariable: 'docker_user')]) {
                        sh "docker login $registryUrl -u ${docker_user} -p ${docker_pass}"
}
                    
                  //  sh 'docker tag  hello:latest $registryUrl/hello:${BUILD_NUMBER}'
                    sh 'docker push $registryUrl/hello:${BUILD_NUMBER}'
                    
                }
            }
            stage( 'Login to AKS repo') {
                steps {
                        sh 'rm -rf *'
                     withCredentials([usernamePassword(credentialsId: 'Argocd', passwordVariable: 'Argopwd', usernameVariable: 'Argocd')]) {
                      //git remote set-url origin https://ksv102:${Argocd}@github.com/ksv102/CD.git
                         sh '''  
                         git config --global user.name "${Argocd}"
                         git config --global user.email "kssarma33@gmail.com"
                         git clone https://${Argopwd}@github.com/ksv102/CD.git
                          '''
                     } 
                }
            }
            
          /*  stage( 'Update to AKS repo') {
                steps {
                    sh '''
                        cd CD/
                         git branch
                         rm -rf deployment.yml
                         cp -r /opt/k8s_deploy/deployment.yml ${WORKSPACE}/CD/
                         sed -i "s|LATESTVERSION|$registryUrl/hello:${BUILD_NUMBER}|g" ${WORKSPACE}/CD/deployment.yml
                         git add deployment.yml
                         git commit -m "Build_number"
                         git push -u origin '''
                    
                }
            } */
        }
}
