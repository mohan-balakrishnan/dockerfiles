pipeline{
    agent any
    stages{
        stage('Docker POS image Build & Deploy'){
            steps{
               sshagent(['azureuser']){
                    
                    sh '''
                        ssh -o StrictHostKeyChecking=no azureuser@YOUR-SERVER-IP '

                            sudo chmod -R +r reports/*
                            # kill post
                            sudo fuser -k 80/tcp
                            sudo fuser -k 3500/tcp
                            
                            docker stop pos-system-web
                            docker rm pos-system-web
                            docker stop pos-system-api
                            docker rm pos-system-api
                            
                            docker rmi pos-system-api  
                            docker rmi pos-system-web
                            
                            docker build -t pos-system-web:latest -f Dockerfileweb .
                            docker run -d -p 80:80 --name pos-system-web pos-system-web:latest /bin/sh -c "nohup yarn dev:web"

                            docker build -t pos-system-api:latest -f Dockerfileapi .
                            docker run -d -p 3500:3500 --name pos-system-api pos-system-api:latest /bin/sh -c "nohup yarn dev:api"
                            exit

                        '
                    '''

               }


            }

        }
        stage('Docker QA Test'){
            steps{
               
               // SSH into the server
               sshagent(['azureuser']){
                    

                    sh '''
                        ssh -o StrictHostKeyChecking=no azureuser@YOUR-SERVER-IP '
                            
                            sudo apt  install awscli -y
                            mkdir reports
                            sudo rm -rf reports/*
                            
                            #docker-compose down
                            #docker-compose up -d
                            
                            docker rm $(docker ps -a -q -f status=exited)
                            
                            docker stop pos-sel-test-container
                            docker rm pos-sel-test-container
                            
                            docker rmi pos-selenium-tests
                            
                            docker build -t pos-selenium-tests:Latest -f posdockerfile .
                            docker run -v $(pwd)/reports:/selenium/reports --name pos-sel-test-container pos-selenium-tests:Latest
                            exit

                        '
                    '''

               }


            }

        }
        stage('Push reports to s3'){
            steps{
               
               // SSH into the server
               sshagent(['azureuser']){
                    

                    sh '''
                        ssh -o StrictHostKeyChecking=no azureuser@YOUR-SERVER-IP '
                            
                            sudo chmod -R +r reports/*
                            aws s3 sync reports/ s3://devops-oct-2023/reports

                        '
                    '''

               }


            }

        }
    }
    post {
        success {
            script {
                def buildNumber = currentBuild.number
                def customSubject = "Pos App Dev build - Test status #${buildNumber} - Successful"
                def customMessage = "Pos App Dev build - Test for build #${buildNumber} that was successful. Reports pushed in s3. URL: https://devops-oct-2023.s3.ap-southeast-1.amazonaws.com/reports/ "

                emailext (
                    subject: customSubject, // Custom subject
                    body: customMessage, // Custom message
                    to: 'yourmailid@gmail.com', // Replace with the recipient's email address
                    from: 'yourmailid@gmail.com', // Replace with your Gmail address
                )
            }
        }
        failure {
            script {
                def buildNumber = currentBuild.number
                def customSubject = "Pos App Dev build - Test #${buildNumber} - Failed"
                def customMessage = "Pos App Dev build - Test for build #${buildNumber} that failed."

                emailext (
                    subject: customSubject, // Custom subject
                    body: customMessage, // Custom message
                    to: 'yourmailid@gmail.com', // Replace with the recipient's email address
                    from: 'yourmailid@gmail.com', // Replace with your Gmail address
                )
            }
        }
    }
}
