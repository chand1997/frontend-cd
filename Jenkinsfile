pipeline{
    agent { label "agent-1" }
    parameters{
        string(name: 'version', description: 'Enter version')
    }
    stages{
    stage("get-target-arn"){
        steps{
            script{
                withAWS(credentials: 'aws-creds'){
                    env.TARGET_ARN = sh(
                    script: """
                            aws elbv2 describe-target-groups \
                             --names expense-dev \
                             --query 'TargetGroups[0].TargetGroupArn' \
                             --output text
                            
                            """,
                            returnStdout: true
                    ).trim()
                }
            }
        }
    }
    stage("frontend-cd"){
       steps{
        script{
            withAWS(credentials: 'aws-creds'){
            sh """
                aws eks update-kubeconfig --region us-east-1 --name expense-dev
                
                helm upgrade --install frontend -n expense -f values.yaml --set imageVersion=${params.version} --set targetARN=${env.TARGET_ARN}  .
                """
            }  
        }
       }
    }

    }
   

}

// use delimeter(|), coz the replacing value targetARN also has "/"
// or
// do below
// 1. change values.yaml file like below
// utilization: 50
// replicas: 1
// imageURL: 353654037274.dkr.ecr.us-east-1.amazonaws.com/expense/frontend
// imageVersion: ""
// targetARN: ""

// 2. use this command
// helm upgrade --install frontend -n expense -f values.yaml --set imageVersion=${params.version} --set targetARN=${env.TARGET_ARN}  .