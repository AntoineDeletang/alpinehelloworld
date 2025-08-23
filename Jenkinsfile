pipeline {
     environment {
       ID_DOCKER = "${ID_DOCKER_PARAMS}"
       IMAGE_NAME = "alpinehelloworld"
       IMAGE_TAG = "latest"
       PORT_EXPOSED = "80"
       STAGING = "${ID_DOCKER}-staging"
       PRODUCTION = "${ID_DOCKER}-production"
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
     stage('Connect to VM') {
    agent any
    environment {
        SSH_KEY = credentials('vm_private_key')
      }
      steps {
        script {
            sh '''
            echo "$SSH_KEY" > /tmp/vm_key
            chmod 600 /tmp/vm_key
            ssh -i /tmp/vm_key -o StrictHostKeyChecking=no vagrant@192.168.1.50 "echo Hello from VM"
            rm /tmp/vm_key
            '''
      }
     }
     }
     stage('Deploy') {
      agent any 
      steps {
        script {
          sh '''
            echo "Cloning repo"
            git clone https://github.com/AntoineDeletang/alpinehelloworld.git
            echo "Building image"
            docker build -t ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG .
            echo "Running container"
            docker run --name $IMAGE_NAME -d -p ${PORT_EXPOSED}:5000 -e PORT=5000 ${ID_DOCKER}/$IMAGE_NAME:$IMAGE_TAG
          '''
        }
      }
     }
  }
} 
