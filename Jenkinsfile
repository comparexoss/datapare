pipeline {
    environment {
        GIT_REPO = "https://github.com/comparexoss/datapare.git"
        ACR_LOGINSERVER = "hakkiogretmen"
        //ACR_REPO = 'mstrdevopsworkshop'
        //ACR_CRED = credentials('dockerhub')
        //WEB_IMAGE="${env.ACR_LOGINSERVER}/${env.ACR_REPO}/rating-web"
        WEB_IMAGE="${env.ACR_LOGINSERVER}/rating-web"
        //API_IMAGE="${env.ACR_LOGINSERVER}/${env.ACR_REPO}/rating-api"
        API_IMAGE="${env.ACR_LOGINSERVER}/rating-api"
        //DB_IMAGE="${env.ACR_LOGINSERVER}/${env.ACR_REPO}/rating-db"
    }
  options { 
      disableConcurrentBuilds() 
      timestamps()
  }
  agent any
  stages {
      stage('Checkout') {
        steps {
            git url: "${env.GIT_REPO}", branch: 'master'
        }
      }
    
        stage('Build images') {
            steps{
                dir('api')
                {
                    script{
                    docker.build("${env.API_IMAGE}:${env.BUILD_NUMBER}")
                    }
                }
                dir('web')
                {
                   sh "docker build --build-arg BUILD_DATE=`date '+%Y-%m-%dT%H:%M:%SZ'` --build-arg VCS_REF=`git rev-parse --short HEAD` --build-arg IMAGE_TAG_REF=${env.BUILD_NUMBER} -t ${env.WEB_IMAGE}:${env.BUILD_NUMBER} ."
                }    
            }
       }
      stage('Test api image') {
         agent { docker "${env.API_IMAGE}:${env.BUILD_NUMBER}" } 
               steps {
                        sh 'echo $HOSTNAME'
                     }
        }
      
       stage('Test web image') {
         agent { docker "${env.WEB_IMAGE}:${env.BUILD_NUMBER}" } 
               steps {
                        sh 'echo $HOSTNAME'
                     }
      }

    stage('Push images to ACR') {
        steps{
            withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub',usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
            sh "docker login -u $USERNAME -p $PASSWORD"
            sh "docker push ${env.API_IMAGE}:${env.BUILD_NUMBER}"
            sh "docker tag ${env.API_IMAGE}:${env.BUILD_NUMBER} ${env.API_IMAGE}:latest"
            sh "docker push ${env.API_IMAGE}:latest"
           
            sh "docker push ${env.WEB_IMAGE}:${env.BUILD_NUMBER}"
            sh "docker tag ${env.WEB_IMAGE}:${env.BUILD_NUMBER} ${env.WEB_IMAGE}:latest"
            sh "docker push ${env.WEB_IMAGE}:latest"
            }
        }
    }
    stage('Delete Old Images'){
          steps{
            sh 'echo "eski imagelari sil"' 
          }
    }
    stage('ACI recreate orchestration') {
        steps{
            sh 'echo "acilari recreate et"' 
        }
    }
}

}