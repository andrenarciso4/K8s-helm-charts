pipeline {
  environment {
        HELM_BUCKET='helm-20170102-8'
        PACKAGE='manager-chart'
        MYCHART='my-charts'

  }
  agent {
    kubernetes {
      label 'helm-pod'
	  serviceAccount 'jenkins-helm'
      containerTemplate {
        name 'helm'
        image 'andrenarciso4/helm-s3'
        ttyEnabled true
        command 'cat'
      }
    }
  }
  stages {
    stage('Preparation helm') {
      steps {
        container('helm') {
          git url: 'https://github.com/andrenarciso4/K8s-helm-charts.git', branch: 'master'
        }
      }
    }

    stage('Build helm') {
      steps {
        container('helm') {
			sh '''
            export AWS_REGION=eu-central-1
			cp -r /home/helm/.helm ~
			helm repo add ${MYCHART} s3://${HELM_BUCKET}/charts
			cd ${PACKAGE}
			helm dependency update
			helm package .
            helm s3 push --force ${PACKAGE}-*.tgz ${MYCHART}
            '''
       }
     }
   }
	stage('Deploy helm') {
      steps {
        container('helm') {
          sh '''
          export AWS_REGION=eu-central-1
          cp -r /home/helm/.helm ~
          helm repo add ${MYCHART} s3://${HELM_BUCKET}/charts
          DEPLOYED=$(helm list |grep -E "^${PACKAGE}" |grep DEPLOYED |wc -l)
          if [ $DEPLOYED == 0 ] ; then
            helm install --name ${PACKAGE} ${MYCHART}/${PACKAGE}
          else
            helm upgrade ${PACKAGE} ${MYCHART}/${PACKAGE}
          fi
          echo "deployed!"
          '''
        }
      }
    }
  }
}

