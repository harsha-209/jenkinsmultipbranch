pipeline {
  agent any
  environment {
    PRE_PROD_INVENTORY = "prod"
    STAGING_INVENTORY = "dev"
 //   PRE_PROD_BRANCH = /^release\/.*/
    PRE_PROD_BRANCH = "release"
    STAGING_BRANCH = "integration"
    INVENTORY_NAME = """${
      BRANCH_NAME ==~ PRE_PROD_BRANCH ? PRE_PROD_INVENTORY : BRANCH_NAME == STAGING_BRANCH ? STAGING_INVENTORY : 'Unknown'
    }"""
    TAG_NAME = """${
      BRANCH_NAME ==~ PRE_PROD_BRANCH ? 'prod' : 'dev'
    }"""
    BUILD_ENV = """${	
      BRANCH_NAME ==~ PRE_PROD_BRANCH ? 'prod' : 	
      BRANCH_NAME == STAGING_BRANCH ? 'dev' : 'prod'	
    }"""
  }
  stages {
    stage('Validate branch') {
      when {
        equals expected: 'Unknown',
        actual: env.INVENTORY_NAME
      }
      steps {
        script {
          currentBuild.result = "ABORTED"
          error "Aborting build as the branch not part of plan"
        }
      }
    }
    stage('Create Docker Image') {
      steps {
        echo "The branch: ${BRANCH_NAME} and the build ${BUILD_NUMBER}"
        sh """
          sudo docker build --build-arg BUILD_ENV=${BUILD_ENV}  -t rummy-server .
          sudo docker tag rummy-server registry-server:8083/rummy-server-${TAG_NAME}:latest
          sudo docker tag rummy-server registry-server:8083/rummy-server-${TAG_NAME}:${BUILD_NUMBER}
        """
      }
    }
    stage('Push to Docker Registry') {
      steps {
        echo 'we will push the image to registry rummy-server-${TAG_NAME}'
        sh """
          sudo docker push registry-server:8083/rummy-server-${TAG_NAME}:latest
          sudo docker push registry-server:8083/rummy-server-${TAG_NAME}:${BUILD_NUMBER}
        """
      }
    }
    stage('Deploy') {
      steps {
        sh """
          sudo ansible-playbook ansible.yml --extra-vars "deployment_host=${INVENTORY_NAME} tag_name=${TAG_NAME}"
        """
      }
    }
  }
  post {
    aborted {
      emailext attachLog: true,
      mimeType: 'text/html',
      body: '${FILE, path="/var/lib/jenkins/mailtemplate/aborted.html"}',
      compressLog: true,
      recipientProviders: [developers(), requestor()],
      subject: '${DEFAULT_SUBJECT}',
      to: '$DEFAULT_RECIPIENTS'
    }
    failure {
      emailext attachLog: true,
      mimeType: 'text/html',
      body: '${FILE, path="/var/lib/jenkins/mailtemplate/failure.html"}',
      compressLog: true,
      recipientProviders: [developers(), requestor()],
      subject: '${DEFAULT_SUBJECT}',
      to: '$DEFAULT_RECIPIENTS'
    }
    success {
      emailext attachLog: true,
      mimeType: 'text/html',
      body: '${FILE, path="/var/lib/jenkins/mailtemplate/success.html"}',
      compressLog: true,
      recipientProviders: [developers(), requestor()],
      subject: '${DEFAULT_SUBJECT}',
      to: '$DEFAULT_RECIPIENTS'
    }
  }
}
