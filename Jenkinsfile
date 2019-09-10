pipeline {
    agent any

    stages {

       stage('SCM Checkout') {
            steps {
                git credentialsId: '8e237d54-cc07-4aad-a3fe-51855a4d84c1', url: 'https://github.com/pramodk05/java_maven_jenkins.git'
            }
        }

        stage('Compile Stage') {
            steps {
                withMaven(maven : 'maven_3.6') {
                    sh 'mvn clean compile'
                }
            }
        }


        stage('Test Stage') {
            steps {
                withMaven(maven : 'maven_3.6') {
                    sh 'mvn test'
                }
            }
        }


        stage('Create the Build artifacts Stage (Package)') {
            steps {
                withMaven(maven : 'maven_3.6') {
                    sh 'mvn package'
                }
            }
        }

        stage ('Generating Tomcat SSH keys') {
            steps {
                sh """
                if [ ! -f ${WORKSPACE}/tomcat_ec2_key ]; then
                    ssh-keygen -f ${WORKSPACE}/tomcat_ec2_key -N ""
                fi
                """
            }
        }

        stage ('Terraform Setup') {
            steps {
                script {
                    def tfHome = tool name: 'Terraform_0.12.6', type: 'org.jenkinsci.plugins.terraform.TerraformInstallation'

                }
            sh 'terraform --version'

            }
        }
        stage ('Terraform Init and Plan') {
            steps {
                sh 'terraform init $WORKSPACE'
                sh 'terraform plan'
            }
        }

        stage ('Terraform Apply') {
            steps {
                sh 'terraform apply --auto-approve'
            }
        }

        stage ('Setting up Host variables') {
            steps {
                sh """
                    terraform output -json tomcat_public_dns | cut -d '"' -f2 > tc_pub_dns.txt
                    terraform output -json tomcat_private_dns | cut -d '"' -f2 > tc_pri_dns.txt
                    terraform output -json tomcat_public_ip | cut -d '"' -f2 > tc_pub_ip.txt
                    terraform output -json tomcat_private_ip | cut -d '"' -f2 > tc_pri_ip.txt
                    sed -n '1p' < /opt/pup_setup_tf/ec2_private_dns.txt > pup_master_pri_dns.txt
                    sed -n '1p' < /opt/pup_setup_tf/ec2_private_ip.txt > pup_master_pri_ip.txt                  
                """
            }
        }    
        
        stage ('Setting up puppet node on Tomcat server') {
            steps {
                sh './tc_pup_agent_setup.sh'
            }
        }

    }
}

