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
        // stage("ssh-test"){
        //     steps{
        //         script{
        //             withCredentials([sshUserPrivateKey(credentialsId: "ssh-build", keyFileVariable: "key")]){
        //                 sh"""ssh -t -i ${key} ubuntu@54.242.61.45
        //                      touch 1.new
        //                 """
        //             }
        //         }
        //     }
        stage("s3-upload"){
            steps{
                script{
                    sh"""
                        aws s3 cp /home/ubuntu/target/*.war s3://bucket-test-for-ec2
                    """
                }
            }
        }
    }
}
