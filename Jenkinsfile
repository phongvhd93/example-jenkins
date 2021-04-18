#!groovy

def lastCommitInfo = ""
def skippingText = ""
def commitContainsSkip = 0
def slackMessage = ""
def shouldBuild = false

def pollSpec = "" 

if(env.BRANCH_NAME == "master") {
  pollSpec = "*/5 * * * *"
} else if(env.BRANCH_NAME == "test") {
  pollSpec = "* * * * 1-5"
}

pipeline {
  agent any

  environment {
        // Fastlane Environment Variables
        PATH = "$HOME/.rvm/gems/ruby-2.7.2/bin" +
                "$HOME/.rvm/rubies/ruby-2.7.2/bin:" +
                "$HOME/.rvm/gems/ruby-2.7.2@global/bin:" +
                "/usr/local/bin:" +
                "$PATH"
    }

  triggers {
    pollSCM ignorePostCommitHooks: true, scmpoll_spec: pollSpec
  }

  stages {
    stage('Init') {
      steps {
        script {
          lastCommitInfo = sh(script: "git log -1", returnStdout: true).trim()
          commitContainsSkip = sh(script: "git log -1 | grep '.*\\[skip ci\\].*'", returnStatus: true)

          if(commitContainsSkip == 0) {
            skippingText = "Skipping commit."
            env.shouldBuild = false
            currentBuild.result = "NOT_BUILT"
          }
        }
      }
    }

    stage('Print fastlane version') {
      steps {
        sh "fastlane --version"
      }
    }

    stage('Build Ad-Hoc') {
      when {
        expression {
          return env.shouldBuild != "false"
        }
      }

      steps {
        sh "fastlane build_adhoc"
      }
    }
  }
}
