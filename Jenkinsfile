stage 'CI'
node {

  checkout scm

  // git branch: 'jenkins2-course', 
  //     url: 'https://github.com/youngl98/solitaire-systemjs-course.git'

  // pull dependencies from npm
  // on windows use: bat 'npm install'
  sh 'npm install'

  // stash code & dependencies to expedite subsequent testing
  // and ensure same code & dependencies are used throughout the pipeline
  // stash is a temporary archive
  stash name: 'everything', 
        excludes: 'test-results/**', 
        includes: '**'
  
  // test with PhantomJS for "fast" "generic" results
  // on windows use: bat 'npm run test-single-run -- --browsers PhantomJS'
  sh 'npm run test-single-run -- --browsers PhantomJS'
  
  // archive karma test results (karma is configured to export junit xml files)
  junit 'test-results/**/test-results.xml'
        
}

// parallel integration testing
stage('Browser Testing') {
  parallel chrome: {
      runTests("Chrome")
  }, safari: {
      runTests("Safari")
  }
}

def runTests(browser) {
  node {
      sh 'rm -rf *'
      unstash 'everything'
      sh "npm run test-single-run -- --browsers ${browser}"
      junit 'test-results/**/test-results.xml'
  }
}

node {
  notify('Deploy to staging?')
}

// limit concurrency so we don't perform simultaneous deploys
// and if multiple pipelines are executing,
// newest is only that will be allowed through, rest will be cancelled
stage ('Deploy') {
  input 'Deploy to staging?'
  milestone()
  
  node {
      // write build number to index page so we can see this update
      sh "echo '<h1>${env.BUILD_DISPLAY_NAME}</h1>' >> app/index.html"
      
      // deploy to a docker container mapped to port 3000
      sh 'docker-compose up -d --build'
      
      notify 'Solitaire Deployed!'
  }
}

def notify(status){
  emailext (
    to: "younglee@visa.com",
    subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
    body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
      <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
  )
}