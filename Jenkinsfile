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
    stage('Run kitchen destroy'){
      steps {
        sh 'kitchen destroy'
      }
    }
    stage('Run kitchen create'){
      steps {
        sh 'kitchen create'
      }
    }
    stage('Run kitchen converge'){
      steps {
        sh 'kitchen converge'
      }
    }
    stage('Run kitchen verify'){
      steps {
        sh 'kitchen verify'
      }
    }
    stage('kitchen destroy'){
      steps {
        sh 'kitchen destroy'
      }
    }
    stage('Create Chef PEM') {
      steps {
        withCredentials([file(credentialsId: 'chef-pem', variable: 'CHEF_PEM')]) {
          sh 'cat > $WORKSPACE/student01.pem < $CHEF_PEM'
        }
      }
    }
    stage('Upload to Chef Infra Server') {
      steps {
        sh 'chef install $WORKSPACE/Policyfile.rb -c $WORKSPACE/config.rb'
        sh 'chef push prod $WORKSPACE/Policyfile.lock.json -c $WORKSPACE/config.rb'
      }
    }
    stage('Converge Chef-managed webserver nodes') {
      steps {
        withCredentials([sshUserPrivateKey(credentialsId: 'agent-creds', keyFileVariable: 'AGENT_KEY', usernameVariable: 'ubuntu')]) {
          sh 'knife ssh "policy_name:apache" -x ubuntu -i $AGENT_KEY "sudo chef-client" -c $WORKSPACE/config.rb'
        }
      }
    }
    stage('Send Slack Notification') {
      steps {
        slackSend color: 'warning', message: "Steph: Please approve ${env.JOB_NAME}${env.BUILD_NUMBER}(<${env.JOB_URL} | Open>)"
      }
    }
    stage('Request Input') {
      steps {
        input 'Please approve or deny this build'
      }
    }
  }
}