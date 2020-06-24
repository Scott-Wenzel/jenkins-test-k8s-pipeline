pipeline {
  agent {
    kubernetes {
      label 'jenkins-test-k8s-pipeline'  // all your pods will be named with this prefix, followed by a unique id
      idleMinutes 5  // how long the pod will live after no jobs have run on it
      yamlFile 'build-pod.yaml'  // path to the pod definition relative to the root of our project 
      defaultContainer 'maven'  // define a default container if more than a few stages use it, will default to jnlp container
    }
  }
  stages {
    stage('Build') {
      steps {  // no container directive is needed as the maven container is the default
        sh "mvn clean package"   
      }
    }
    stage('Build Docker Image') {
      steps {
        container('docker') {
          gitCommit = findGitCommit()
          dockerTag = "${env.BUILD_NUMBER}-${gitCommit}"
          withCredentials([usernamePassword(credentialsId: '68527666-ec6d-4c9a-afb2-a9e7e9e78431', usernameVariable: 'USER', passwordVariable: 'TOKEN')]) {
            sh "docker login -u $USER -p $TOKEN"  // Login   
            sh "docker build -t jwenzel/jenkins-test-k8s-pipeline:dev ."  // when we run docker in this step, we're running it via a shell on the docker build-pod container,
            sh "docker push jwenzel/jenkins-test-k8s-pipeline:${dockerTag}"        // which is just connecting to the host docker deaemon
          }
        }
      }
    }
    stage('Helm Deployment') {
      steps {
        container('helm') {
          withCredentials([file(credentialsId: 'kube-config', variable: 'kubecfg')]){
            //Deploy with Helm
            echo "Deploying to test"
            sh("helm upgrade --install --force jenkins-test-k8s-pipeline --namespace test --set image.tag=${dockerTag} ./helm")
          }
        }
      }
    }
  }
}
