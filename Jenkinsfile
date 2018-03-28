pipeline {
  agent any
  environment {
      tag = VersionNumber (versionNumberString: '${BUILD_DATE_FORMATTED, "yyyyMMdd"}-ci-${BUILDS_TODAY}')
  }
  stages {
    stage('build') {
      when { branch 'ci' }
      steps {
        sh '''ls -al ${WORKSPACE}
docker run --rm -v ${WORKSPACE}:/docs squidfunk/mkdocs-material build
cd ${WORKSPACE}
ls -alR site
cat - > Dockerfile <<EOF
FROM nginx
COPY site/ /usr/share/nginx/html/
EOF

docker rm safespring-docs:$tag || /bin/true
docker build -t registry.example.safespring.com/safespring/docs:$tag .
'''
      }
      post {
        failure {
          cleanWs()
        }
      }
    }
    stage('test') {
      when { branch 'ci' }
      steps {
        sh '''docker run --rm --name safespring-docs -d -p 8580:80 registry.example.safespring.com/safespring/docs:$tag
curl localhost:8580 > /dev/null
docker stop safespring-docs'''
      }
      post {
        failure {
          cleanWs()
        }
      }
    }
    stage('push') {
      when { branch 'ci' }
      steps {
        sh 'docker push registry.example.safespring.com/safespring/docs:$tag'
      }
      post {
        failure {
          cleanWs()
        }
      }
    }
    stage('k8s-deploy') {
      when { branch 'ci' }
      steps {
        sh 'sed -i "s/%%TAG%%/$tag/g" k8s.yaml'
        sh 'kubectl apply -f k8s.yaml'
        sh 'kubectl describe deployment safespring-docs-deployment'
        sh 'kubectl describe service safespring-docs'
      }
      post {
        always {
          cleanWs()
        }
      }
    }
  }
}
