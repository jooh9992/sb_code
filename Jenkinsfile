pipeline {
  agent any
  // any, none, label, node, docker, dockerfile, kubernetes
  tools {
    maven 'my_maven'
  }

  environment {
    gitName = 'jooh9992'
    gitEmail = 'jooh9992@gmail.com'
    githubCredential = 'git_cre'
    dockerHubRegistry = 'leejooh/sbimage'
    dockerHubRegistryCredential = 'docker-cre'
  }
  stages {
    stage('Checkout Github') {
      steps {
          checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: githubCredential, url: 'https://github.com/jooh9992/sb_code']]])
          }
      post {
        failure {
          echo 'Repository clone failure'
        }
        success {
          echo 'Repository clone success'  
        }
      }
    }

    stage('Gradle Build') {
      steps {
          sh 'gradle clean build'
          }
      post {
        failure {
          echo 'Gradle jar build failure'
        }
        success {
          echo 'Gradle jar build success'  
        }
      }
    }
    stage('Docker Image Build') {
      steps {
          sh "docker build -t ${dockerHubRegistry}:${currentBuild.number} ."
          sh "docker build -t ${dockerHubRegistry}:latest ."
          }
      post {
        failure {
          echo 'Docker image build failure'
        }
        success {
          echo 'Docker image build success'  
        }
      }
    }
    stage('Docker Image Push') {
      steps {
          // 도커 허브의 크리덴셜
          withDockerRegistry(credentialsId: dockerHubRegistryCredential, url: '') {
          // withDockerRegistry : docker pipeline 플러그인 설치시 사용가능.
          // dockerHubRegistryCredential : environment에서 선언한 docker_cre  
            sh "docker push ${dockerHubRegistry}:${currentBuild.number}"
            sh "docker push ${dockerHubRegistry}:latest"
          }  
      }
      post {
        failure {
          echo 'Docker Image Push failure'
          sh "docker rmi ${dockerHubRegistry}:${currentBuild.number}"
          sh "docker rmi ${dockerHubRegistry}:latest"
        }
        success {
          echo 'Docker Image Push success'
          sh "docker rmi ${dockerHubRegistry}:${currentBuild.number}"
          sh "docker rmi ${dockerHubRegistry}:latest"
        }
      }
    }
    stage('k8s manifest file update') {
      steps {
                  git credentialsId: githubCredential,
                  url: 'https://github.com/jooh9992/sb_code.git',
                  branch: 'main'  

                  // 이미지 태그 변경 후 메인 브랜치에 푸시
                  sh "git config --global user.email ${gitEmail}"
                  sh "git config --global user.name ${gitName}"
                  sh "sed -i 's/sbimage:.*/sbimage:${currentBuild.number}/g' deploy/sb-deploy.yml"
                  // deploy폴더의 sd-deploy.yml 파일의 내용을 수정하는 부분.
                  sh "git add ."
                  sh "git commit -m 'fix:${dockerHubRegistry} ${currentBuild.number} image versioning'"
                  sh "git branch -M main"
                  sh "git remote remove origin"
                  sh "git remote add origin git@github.com:jooh9992/sb_code.git"
                  sh "git push -u origin main"

          }
      post {
        failure {
          echo 'Container Deploy failure'
        }
        success {
          echo 'Container Deploy success' 
        }
      }
    }

  }
}
