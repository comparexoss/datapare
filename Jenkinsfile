pipeline {
    environment {
        TENANT_ID="d1b3fd06-9dad-4353-b489-be3e10a99364"
        GIT_REPO = "https://github.com/comparexoss/datapare.git"
        ACR_LOGINSERVER = "hakkiogretmen"
        //ACR_REPO = 'mstrdevopsworkshop'
        ACR_CRED = credentials('dockerhub')
        //WEB_IMAGE="${env.ACR_LOGINSERVER}/${env.ACR_REPO}/rating-web"
        WEB_IMAGE="${env.ACR_LOGINSERVER}/rating-web"
        //API_IMAGE="${env.ACR_LOGINSERVER}/${env.ACR_REPO}/rating-api"
        API_IMAGE="${env.ACR_LOGINSERVER}/rating-api"
        //use that config to remove old images
        IMAGE_AGE=15
        //use that configs to scale container instances. Pass these variables to python script
        WEB_CNTNR_COUNT=15
        API_CNTNR_COUNT=5

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
                    //docker.tag("${env.API_IMAGE}:latest")
                    }
                    sh "docker tag ${env.API_IMAGE}:${env.BUILD_NUMBER} ${env.API_IMAGE}:latest"
                }
                dir('web')
                {
                   sh "docker build --build-arg BUILD_DATE=`date '+%Y-%m-%dT%H:%M:%SZ'` --build-arg VCS_REF=`git rev-parse --short HEAD` --build-arg IMAGE_TAG_REF=${env.BUILD_NUMBER} -t ${env.WEB_IMAGE}:${env.BUILD_NUMBER} ."
                   sh "docker tag ${env.WEB_IMAGE}:${env.BUILD_NUMBER} ${env.WEB_IMAGE}:latest"
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
            withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: "${env.ACR_CRED}",usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
            sh "docker login -u $USERNAME -p $PASSWORD"

            sh "docker push ${env.API_IMAGE}:${env.BUILD_NUMBER}"
            sh "docker push ${env.API_IMAGE}:latest"
           
            sh "docker push ${env.WEB_IMAGE}:${env.BUILD_NUMBER}"
            sh "docker push ${env.WEB_IMAGE}:latest"
            }
        }
    }
    stage('Delete Old Images'){
          steps{
               dir('scripts')
                {
                    sh "chmod 755 clean_docker_images"
                    sh "./clean_docker_images ${env.BUILD_NUMBER} ${env.IMAGE_AGE} ${env.WEB_IMAGE}" 
                    sh "./clean_docker_images ${env.BUILD_NUMBER} ${env.IMAGE_AGE} ${env.API_IMAGE}" 
                }
          }
    }
    stage('ACI recreate orchestration') {
        steps{
               dir('scripts')
                {
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: "${env.azure-credentials}",usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {   
                        sh "chmod 755 aci_orchestration"         
                        sh "./aci_orchestration 'dp_comparex' 'acitest' ${env.WEB_IMAGE} ${env.WEB_CNTNR_COUNT} 'Linux' 'West Europe' 'Always' 'Public' $USERNAME $PASSWORD ${env.TENANT_ID} "
                    }
                }
        }
    }
}

}