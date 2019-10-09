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
        // stage("ssh-test"){
        //     steps{
        //         script{
        //             withCredentials([sshUserPrivateKey(credentialsId: "ssh-build", keyFileVariable: "key",  usernameVariable: 'userName')]){
        //                 remote.user = userName
        //                 remote.identityFile = key
        //                 sshCommand remote: remote, command: 'java -jar -Dspring.profiles.active=test ~/target/spring-boot-rest-example-0.5.0.war &'
        //             }
        //         }
        //     }
        // }
        stage("s3-upload"){
            steps{
                script{
                    sh"""
                        aws s3 cp /var/lib/jenkins/workspace/java-build/target/*.war s3://bucket-test-for-ec2
                    """
                }
            }
        }
        stage("ASG-increase"){
            steps{
                script{
                    sh"""
                        aws autoscaling set-desired-capacity --desired-capacity 4 --auto-scaling-group-name jenkins-test --region us-east-1
                    """
                }
            }
        }
        stage("health-check"){
            steps{
                script{
                    sh"""
                        aws elbv2 describe-target-health  --target-group-arn arn:aws:elasticloadbalancing:us-east-1:159714198409:targetgroup/jenkins-test/236f8f6029b6913a --region us-east-1

                    """

                }
            }
        }

    }
}
