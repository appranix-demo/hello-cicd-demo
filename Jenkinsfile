node {
   stage('Build') {
         git 'https://github.com/appranix-demo/hello-cicd-demo.git/'
         sh './mvnw package'
         archive 'target/*.war'
   }

   stage('Publish to nexus') {
      nexusArtifactUploader artifacts: [[
                                         artifactId: 'quick-demo-app', classifier: '',
                                         file: 'target/quick-demo-app-1.0.0.war', type: 'war'
                                       ]],
         credentialsId: 'nexus',
         groupId: 'com.appranix',
         nexusUrl: 'repo.appranix.net/nexus',
         nexusVersion: 'nexus2',
         protocol: 'https',
         repository: 'releases',
         version: env.BUILD_ID
   }

   stage('Deploy to Prana') {
      withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'customer-demo',
                            usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                //available as an env variable, but will be masked if you try to print it out any which way
               sh 'prana auth logout'
               sh 'echo successfully logged out'

               sh "prana auth login --username=${env.USERNAME} --password=${env.PASSWORD} --account=demo-inc"
               sh 'echo successfully logged in'

               sh "prana config set organization=demo-inc -g"
               sh 'echo organization is set as demo-inc'

               sh "prana config set assembly=ci-demoapp -g"
               sh 'echo assembly is set as ci-demoapp'

               sh "prana design load design.yml"
               sh 'echo design is uploaded'

               sh "prana design variable update -a ci-demoapp --platform=webapp appVersion=${env.BUILD_ID}"

               sh "prana design commit init-commit"
               sh 'echo new design in committed with message init-commit'

               sh "prana transition pull -e prod"
               sh 'echo design pull to prod appspace'

	       sh 'sleep 20'

               sh "prana transition commit init-commit -e prod"

               sh "prana transition deployment create -e prod"
               sh 'echo deployement is started'
      }
   }
}
