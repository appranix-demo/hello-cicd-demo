node {
   stage('Build') {
         git 'https://github.com/veereshwaran/hello-world.git/'
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
                sh 'prana config set site=https://app.appranix.net/web/ -g'
                sh 'prana auth logout'
                sh "prana auth login --username=${env.USERNAME} --password=${env.PASSWORD} --account=devorg"
                sh "prana config set organization=devorg-veeresh -g"
                sh "prana config set assembly=demo2 -g"
                sh "prana design load design.yml"
                sh "prana design variable update -a demo2 --platform=helloworld appVersion=${env.BUILD_ID}"
                sh "prana design commit init-commit"
                sh "prana transition pull -e dev"
                sh "prana transition commit init-commit -e dev"
                sh "prana transition deployment create -e dev"
            }
   }
}
