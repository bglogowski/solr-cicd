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
        httpRequest(
          url: 'https://www.apache.org/dyn/closer.lua/solr/solr/10.0.0/solr-10.0.0-src.tgz?action=download',
          outputFile: 'solr-10.0.0-src.tgz',
          )
     }

   }

  }
}
