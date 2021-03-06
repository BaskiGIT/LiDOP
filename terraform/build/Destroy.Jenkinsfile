#!groovy

pipeline {
  agent {
    label 'default'
  }
  
  options {
    buildDiscarder(logRotator(numToKeepStr: '100', daysToKeepStr: '365'))
  }

  parameters {
    string(name: 'Name', defaultValue: 'Name', description: 'Name of LiDOP')
    string(name: 'User', defaultValue: 'lidop', description: 'Username')
    password(name: 'Password', defaultValue: 'lidop', description: 'Password')
    credentials(name: 'AWS_Key', description: 'AWS_Key', defaultValue: '', credentialType: "org.jenkinsci.plugins.plaincredentials.impl.StringCredentialsImpl", required: true )
    credentials(name: 'AWS_Secret', description: 'AWS_Secret', defaultValue: '', credentialType: "org.jenkinsci.plugins.plaincredentials.impl.StringCredentialsImpl", required: true )  
  }

  environment {
    BuildName = "Create-Stack-${params.Name}.${BUILD_NUMBER}"
  }

  stages {
    stage('Build Number') {
      steps {
        script {
          currentBuild.displayName = "${BuildName}"
        }
      }
    }

    stage("Terraform init") {
      steps {
        sh 'ln -sf ./../terraform.backend.consul.tf ./aws/terraform.tf'
        sh "terraform init -backend-config=address=${env.IPADDRESS}:8500 ./aws"
      }
    }

    stage("Terraform select workspace") {
      steps {
       sh "terraform workspace select ${params.Name} ./aws"
      }
    }

    stage("Terraform plan") {
      steps {
	      withCredentials([string(credentialsId: "${params.AWS_Secret}", variable: 'AWS_SECRET'), string(credentialsId: "${params.AWS_Key}", variable: 'AWS_KEY')]) {
	        sh "terraform plan -var access_key=${AWS_Key} -var enabled=0 -var lidop_name=${params.Name} -var secret_key=${AWS_Secret} -var user_name=${params.User} -var password=${params.Password} ./aws"
        }      
      }
    }

    stage("Terraform destroy") {
      steps {
    	  withCredentials([string(credentialsId: "${params.AWS_Secret}", variable: 'AWS_SECRET'), string(credentialsId: "${params.AWS_Key}", variable: 'AWS_KEY')]) {
	        sh "terraform destroy -auto-approve -var access_key=${AWS_Key} -var enabled=0 -var lidop_name=${params.Name} -var secret_key=${AWS_Secret} -var user_name=${params.User} -var password=${params.Password} ./aws"
        }      
      }
    }

    stage("Terraform delete workspace") {
      steps {
       sh "terraform workspace select default ./aws"
       sh "terraform workspace delete ${params.Name} ./aws"
      }
    }

  }
}
