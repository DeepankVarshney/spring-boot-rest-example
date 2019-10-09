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
                    #!/bin/bash

                    health="healthy"
                    reg="us-east-1"
                    while true; do
                            count=0
                            aws elbv2 describe-target-health  --target-group-arn arn:aws:elasticloadbalancing:us-east-1:159714198409:targetgroup/jenkins-test/236f8f6029b6913a --region $reg | grep "healthy" > state_check.txt
                            input="/home/ubuntu/state_check.txt"
                            if [[ $(wc -l state_check.txt | awk '{print $1}') == 4 ]]; then
                                    while IFS= read -r line
                                    do
                                            a=$(echo $line | cut -d':' -f 2 | cut -d'"' -f 2)
                                            if [[ $a = $health ]]; then
                                                    count=`expr $count + 1`
                                            fi
                                    done < $input
                                    if [[ $count -eq 4 ]]; then
                                            break
                                    fi
                            fi
                    done
                    """

                }
            }
        }

    }
}
