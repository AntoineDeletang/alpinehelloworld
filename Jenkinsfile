/* import shared library */
@Library('shared-library')_

pipeline {
     environment {
       ID_DOCKER = "wawacosy"
       IMAGE_NAME = "alpinehelloworld"
       IMAGE_TAG = "latest"
       PORT_EXPOSED = "80"
     }
     agent none
     stages {
         stage('Build image') {
             agent any
             steps {
                script {
                  sh 'docker build -t ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG .'
                }
             }
        }
        stage('Run container based on builded image') {
            agent any
            steps {
               script {
                 sh '''
                    echo "Clean Environment"
                    docker rm -f $IMAGE_NAME || echo "container does not exist"
                    docker run --name $IMAGE_NAME -d -p ${PORT_EXPOSED}:5000 -e PORT=5000 ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG
                    sleep 5
                 '''
               }
            }
       }
       stage('Test image') {
           agent any
           steps {
              script {
                sh '''
                    curl http://172.17.0.1:${PORT_EXPOSED} | grep -q "Hello world!"
                '''
              }
           }
      }
      stage('Clean Container') {
          agent any
          steps {
             script {
               sh '''
                 docker stop $IMAGE_NAME
                 docker rm $IMAGE_NAME
               '''
             }
          }
     }
     stage('Connect to VM and deploy') {
    agent any
      steps {
        script {
            sh """
            ssh -i ~/.ssh/jenkins_vm_key -o StrictHostKeyChecking=no vagrant@192.168.1.50 "
            echo 'Cloning repo'
            git clone https://github.com/AntoineDeletang/alpinehelloworld.git
            echo 'Building image'
            cd alpinehelloworld
            docker build -t ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG .
            echo 'Running container'
            docker stop $IMAGE_NAME
            docker rm $IMAGE_NAME
            docker run --name $IMAGE_NAME -d -p ${PORT_EXPOSED}:5000 -e PORT=5000 ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG 
            "
            """
      }
     }
    }
  }
    post {
    always {
      script {
        slackNotifier currentBuild.result
      }
    }  
  }    
} 
