node{
      def dockerImageName= 'knackc123/javadedockerapp_$JOB_NAME:$BUILD_NUMBER'
     
      stage('SCM Checkout'){
         git 'https://github.com/KnackCloud/java-groovy-docker.git'
      }
      stage('Build'){
         // Get maven home path and build
         def mvnHome =  tool name: 'Maven 3.5.4', type: 'maven'   
         sh "${mvnHome}/bin/mvn package -Dmaven.test.skip=true"
      }       
     
     stage ('Test'){
         def mvnHome =  tool name: 'Maven 3.5.4', type: 'maven'    
         sh "${mvnHome}/bin/mvn verify; sleep 3"
      }
      
     stage('Build Docker Image'){         
           sh "docker build -t ${dockerImageName} ."
      }  
   
      stage('Publish Docker Image'){
         withCredentials([string(credentialsId: 'dockerpwdkc', variable: 'dockerPWD')]) {
               sh "docker login -u knackc123 -p ${dockerPWD}"
         }
        sh "docker push ${dockerImageName}"
      }
      
    stage('Run Docker Image'){
            def dockerContainerName = 'javadedockerapp_$JOB_NAME_$BUILD_NUMBER'
            def changingPermission='sudo chmod +x /home/ec2-user/stopscript.sh'
            def scriptRunner='sudo /home/ec2-user/stopscript.sh'           
            def dockerRun= "sudo docker run -p 8082:8080 -d --name ${dockerContainerName} ${dockerImageName}" 
            withCredentials([string(credentialsId: 'deploymentserverpwd', variable: 'dpPWD')]) {
                  sh "sshpass -p ${dpPWD} ssh -o StrictHostKeyChecking=no root@34.207.156.212" 
                  sh "sshpass -p ${dpPWD} scp -r stopscript.sh root@34.207.156.212:/home/ec2-user" 
                  sh "sshpass -p ${dpPWD} ssh -o StrictHostKeyChecking=no root@34.207.156.212 ${changingPermission}"
                  sh "sshpass -p ${dpPWD} ssh -o StrictHostKeyChecking=no root@34.207.156.212 ${scriptRunner}"
                  sh "sshpass -p ${dpPWD} ssh -o StrictHostKeyChecking=no root@34.207.156.212 ${dockerRun}"
            }
            
      
      }
       stage('Performance Test'){
            sh "sudo su"
            sh "cd /home/ec2-user/jmeter/apache-jmeter-5.4.1/bin"
            sh "sh /home/ec2-user/jmeter/apache-jmeter-5.4.1/bin/jmeter.sh -n -t /home/ec2-user/jmeter/apache-jmeter-5.4.1/bin/demo1.jmx -l /home/ec2-user/jmeter/apache-jmeter-5.4.1/bin/report.jtl"

      }
      
         
  }
      
