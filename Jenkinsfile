def remote = [:]
remote.name = "app-server"
remote.host = "54.236.9.207"
remote.allowAnyHosts = true

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
                    withCredentials([sshUserPrivateKey(credentialsId: "ssh-build", keyFileVariable: "key",  usernameVariable: 'userName')]){
                        remote.user = userName
                        remote.identityFile = key
                        sshCommand remote: remote, command: 'java -jar -Dspring.profiles.active=test ~/target/spring-boot-rest-example-0.5.0.war &' 
                    }
                }
            }
        stage("s3-upload"){
            steps{
                script{
                    sh"""
                        aws s3 cp /var/lib/jenkins/workspace/test_pipe/target/*.war s3://bucket-test-for-ec2
                    """
                }
            }
        }
    }
}
