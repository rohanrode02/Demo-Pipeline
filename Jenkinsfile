pipeline {
  agent any

  options {
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '15'))
  }

  parameters {
    choice(name: 'ENV', choices: ['dev', 'qa', 'prod'], description: 'Choose target environment')
  }

  triggers {
    // Optional if you also wire a GitHub webhook:
    pollSCM('H/2 * * * *') // check every ~2 minutes
  }

  stages {
    stage('Checkout') {
      steps {
        // PUBLIC repo:
        git url: 'https://github.com/rohanrode02/Demo-Pipeline.git', branch: 'main'

        // If your repo is PRIVATE, comment the line above and uncomment below:
        // git branch: 'BRANCH_HERE', url: 'REPO_URL_HERE', credentialsId: 'CREDENTIALS_ID_HERE'
      }
    }

    stage('Build') {
      steps {
        bat '''
        echo === BUILD START ===
        echo Date: %date% %time%
        if not exist build mkdir build
        echo Sample artifact created by Jenkins on %date% %time% > build\\artifact.txt
        echo === BUILD END ===
        '''
      }
    }

    stage('Archive Artifact') {
      steps {
        archiveArtifacts artifacts: 'build\\artifact.txt', onlyIfSuccessful: true
      }
    }

    stage('Deploy') {
      when { expression { params.ENV in ['dev','qa','prod'] } }
      steps {
        bat """
        set TARGET_DIR=C:\\deploy\\%ENV%
        if not exist %TARGET_DIR% mkdir %TARGET_DIR%
        copy /Y build\\artifact.txt %TARGET_DIR%\\artifact-%ENV%.txt
        echo Deployed artifact to %TARGET_DIR%
        """
      }
    }
  }

  post {
    success {
      echo "✅ ${env.JOB_NAME} #${env.BUILD_NUMBER} finished OK (ENV=${params.ENV})"
    }
    failure {
      // Requires Email Extension plugin + SMTP configured
      emailext(
        subject: "❌ Jenkins FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        to: 'your_email@example.com',
        body: """Build failed.

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
ENV: ${params.ENV}
Console: ${env.BUILD_URL}console
"""
      )
    }
  }
}
