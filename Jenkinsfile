pipeline{
    agent any

    stages{
        stage("git pull"){
            steps{
                script{
                    git branch: "master", changelog: false, url: "https://github.com/DeepankVarshney/spring-boot-rest-example.git"
                }
            }
        }
        stage("maven-build"){
            steps{
                script{
                    withMaven(maven: "mavenTool")
                    {
                    sh"""
                        mvn clean package
                    """
                    }
                }
            }
        }
        stage("ssh-test"){
            steps{
                script{
                    withCredentails([sshUserPrivateKey(credentialsId: "ssh-build", keyFileVariable: "key")]){
                        sh"""ssh -i ${key} ubuntu@54.236.9.207
                             touch 1.new
                        """
                    }
                }
            }
        }
    }
}
