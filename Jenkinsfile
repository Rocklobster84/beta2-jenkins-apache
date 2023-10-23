pipeline {
  agent { label "agentfarm" }
  stages {
    stage('Delete the workspace') {
      steps {
        cleanWs()
      }
    }
    stage('Installing Chef Workstation') {
      steps {
        script {
          def exists = fileExists '/usr/bin/chef-client'
          if (exists == true) {
            echo "Skipping Chef Workstation install - already installed"
          } else {
            sh 'sudo apt-get install -y wget tree unzip'
            sh 'wget https://packages.chef.io/files/stable/chef-workstation/23.5.1040/ubuntu/22.04/chef-workstation_23.5.1040-1_amd64.deb'
            sh 'sudo dpkg -i chef-workstation_23.5.1040-1_amd64.deb'
            sh 'sudo chef env --chef-license accept'
          }
        }
      }
    }
    stage('Download Apache Cookbook') {
      steps {
        git branch: 'main', credentialsId: 'git-repo-creds', url: 'git@github.com:Rocklobster84/beta2-jenkins-apache.git'
      }
    }
    stage('Install Kitchen Docker Gem') {
      steps {
        sh 'sudo apt-get install -y make gcc'
        sh 'sudo chef gem install kitchen-docker'
      }
    }
  }
}