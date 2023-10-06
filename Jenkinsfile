pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }
    stages {
         stage('CleanWorkspace') {
            steps {
                cleanWs()
            }
         }
         stage('Clone repository') { 
            steps { 
                script{
               checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/vignesh-vicky-tech/source-repo.git']]])
                }
            }
        }
          stage('Junit Test & Security Tests'){
            steps {
                 echo 'Empty'
            }
        }

        stage('Docker Build') { 
            steps { 
                script{
                 sh 'cd ${WORKSPACE}/src/app/'
                 app = docker.build("jenkins_pipeline", "-f ${WORKSPACE}/src/app/Dockerfile .")
                }
            }
            
        }
      
        stage('Docker Image push into ECR') {
            steps {
                script{
                    docker.withRegistry('https://public.ecr.aws/u3o4u3t7/jenkins-public', 'ecr:ap-southeast-2:aws-credentials') {
                    app.push("${env.BUILD_NUMBER}")
                    app.push("latest")
                    }
                }
            }
        }
    
      
         stage('Updating HELM Charts') { 
             
              environment {
                HELM_GIT_REPO_URL = "https://github.com/vignesh-vicky-tech/configuration-repo.git"
                GIT_REPO_EMAIL = 'vigneshs1711@gmail.com '
                GIT_REPO_BRANCH = "main"
                TAG_ID = "${env.BUILD_NUMBER}"
                GIT_USER = 'vignesh-vicky-tech'
                GIT_CREDS = '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'
       
             }
      
      steps {
          cleanWs()
           script{
               checkout([$class: 'GitSCM', branches: [[name: "*/${GIT_REPO_BRANCH}"]], extensions: [], userRemoteConfigs: [[url: "${HELM_GIT_REPO_URL}"]]])
                }
                
           sh '''#!/bin/bash
            
              git checkout -b ${GIT_REPO_BRANCH}
              sudo yq eval '.deployment.container.image.tag = "'$TAG_ID'"' -i $WORKSPACE/deploy/helm/hello-kubernetes/values.yaml
              
              git add $WORKSPACE/deploy/helm/hello-kubernetes/values.yaml 
              git commit -m 'Triggered Build'
             
              git push https://$GIT_USER:$GIT_CREDS@github.com/vignesh-vicky-tech/configuration-repo.git
            
            '''
             sleep(time: 5, unit: "MINUTES")
                }
            }
        
         stage('Test the Latest Application tag') {
            environment {
                 TAG_ID = "${env.BUILD_NUMBER}"
            }
          steps{
              sh ''' #!/bin/bash
                tag_value=$(curl -L "http://ab5030b49b30a4b2eb5cb2b594dedeb6-1522944986.us-east-1.elb.amazonaws.com/" | grep 'public.ecr.aws/u3o4u3t7/jenkins-public:' | awk '{print $1}' | awk -F':' '{print $2}')
                set -x
               echo tag_value
               echo $TAG_ID
               
                if [ $tag_value = $TAG_ID ]
                then
                     echo "Application is updated !!!"
                else
                     echo "Application is not updated with the  latest tag"
                    exit 1
                fi
                '''
            }
    }
            
          
        }
       
    }
