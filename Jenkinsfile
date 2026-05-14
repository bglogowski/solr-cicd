pipeline 
{ 
  agent any

  options {
        disableConcurrentBuilds()
  }

  stages {

   stage('Get solr-cicd GitHub Repo')
   {
      steps {
        echo "Getting Exploratory Testing Repo"
        git(
        url:'git@github.com:bglogowski/solr-cicd.git',
        credentialsId: 'bglogowski',
        branch: "main"
        )
     }

   }

  }
}
