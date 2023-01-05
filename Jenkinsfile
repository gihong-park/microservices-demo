pipeline{
  agent any

  environment{
    HOME = '.'
  }

  options {
    disableConcurrentBuilds()
  }

  stages {
      // 레포지토리를 다운로드 받음
      stage('Prepare') {
        agent any
        steps {
            echo 'Clonning Repository'

            checkout scm
        }
      }
      stage('Build Docker Image') {
        agent any

        steps {
          echo 'Build&Push Image'

          script {
            def folders = sh(returnStdout: true, script: "ls ./src").split().each
            folders.each { item ->
                def app = docker.build("hongpark/${item}", "./src/${item}")
                docker.withRegistry('https://registry.hub.docker.com', 'docker-credential') {
                  app.push("${env.BUILD_NUMBER}")
                  app.push("latest")
                }
            }
          }
        }
      }
      stage('helm Manifest Update') {
        steps {
          git credentialsId: 'git-credential',
              url: 'https://github.com/gihong-park/helm_manifest',
              branch: 'main'

          sshagent(credentials: ['git-ssh-credential']) {
            sh "helm template micro-service . --set images.tag=${env.BUILD_NUMBER} > kubernetes-manifests/kubernetes.yaml"
            sh "git add values.yaml"
            sh "git commit -m '[UPDATE] checkoutservice ${env.BUILD_NUMBER} image versioning'"
            sh "git push git@github.com:gihong-park/helm_manifest.git"

          }
        }
      }
  }
}
