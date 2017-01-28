node {
   stage('Build') {
         git 'https://github.com/appranix-demo/hello-cicd-demo.git/'
         sh 'mvn install'
         archive 'target/*.war'
   }

   stage('Publish to nexus') {
      nexusArtifactUploader artifacts: [[
                                         artifactId: 'quick-demo-app', classifier: '',
                                         file: 'target/quick-demo-app-1.0.0.war', type: 'war'
                                       ]],
         credentialsId: 'nexus',
         groupId: 'com.appranix',
         nexusUrl: 'i00039.hosts.appranix.info:8081/nexus',
         nexusVersion: 'nexus2',
         protocol: 'http',
         repository: 'releases',
         version: env.BUILD_ID
   }

   stage('Deploy to Prana') {
      withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'ci-appranix',
                            usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                //available as an env variable, but will be masked if you try to print it out any which way
                sh 'prana auth logout'
                sh "prana auth login --username=${env.USERNAME} --password=${env.PASSWORD} --account=testorg"
                sh "prana config set organization=testorg-app-associates -g"
                sh "prana config set assembly=quick-demo-app -g"
                sh "prana design load design.yml"
                sh "prana design variable update -a quick-demo-app --platform=webapp appVersion=${env.BUILD_ID}"
                sh "prana design commit init-commit"
                sh "prana transition pull -e ci"
                sh "prana transition commit init-commit -e ci"
                sh "prana transition deployment create -e ci"
            }
   }
}
