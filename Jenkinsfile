pipeline{
    agent { label "agent-1" }
    parameters{
        string(name: 'version', description: 'Enter version')
         choice(name: 'env', choices: ['dev', 'qa', 'prod'], description: 'Pick environment')
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
                sed -i 's/IMAGE_VERSION/${params.version}/g' values.yaml
                sed -i 's|TARGET_ARN|${env.TARGET_ARN}|g' values.yaml
                helm upgrade --install frontend -n expense -f values.yaml .
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