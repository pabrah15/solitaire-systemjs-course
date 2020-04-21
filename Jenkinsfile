stage 'CI'
node {
    checkout scm
    //git branch: 'jenkins2-course', 
        //url: 'https://github.com/pabrah15/solitaire-systemjs-course.git'

    // pull dependencies from npm
    // on windows use: bat 'npm install'
    sh 'npm install'

    // stash code & dependencies to expedite subsequent testing
    // and ensure same code & dependencies are used throughout the pipeline
    // stash is a temporary archive
    stash name: 'everything', 
          excludes: 'test-results/**', 
          includes: '**'


}


//parallel integration testing
stage 'Browser Testing'

//parallel PhantomJS: { 
//    runTests("PhantomJS")
//},chrome: { 
//   runTests("google-chrome")
//},firefox: { 
//    runTests("firefox")
//}

parallel PhantomJS: { 
   runTests("PhantomJS")
}

node {
    notify("Depoly waiting")

  }

input 'Deploy to statging?'

stage name: 'Deploy', concurrency: 1

node {
    
    //write build number
    sh "echo '<h1>${env.BUILD_DISPLAY_NAME}/h1>' >> app/index.html"
    // deploy to docker container mapped to port 3000
    sh 'docker-compose up -d --build'
    //sh 'docker-compose --build'
    notify 'Solitaire Deployed'
}




def runTests(browser){
  node {
    //sh 'ls'
    sh 'rm -rf'
    unstash 'everything'
    // on windows use: bat 'npm run test-single-run -- --browsers PhantomJS'
    sh "npm run test-single-run -- --browsers ${browser}"
    //sh 'ls'
    // archive karma test results (karma is configured to export junit xml files)
    step([$class: 'JUnitResultArchiver', 
          testResults: 'test-results/**/test-results.xml'])
    
  }
}


def notify(status){
    emailext (
      to: "wesmdemos@gmail.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}
