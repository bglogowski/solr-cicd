pipeline 
{ 
  agent any

  options {
        disableConcurrentBuilds()
  }

stages {

   stage('Get Blue-Green Deployment Repo')
   {
      steps {
        echo "Getting Exploratory Testing Repo"
        git(
        url:'git@github.com:bglogowski/petclinic-blue-green.git',
        credentialsId: '993c4ceb-d3c7-4a1d-9788-3d753a462b4a',
        branch: "main"
        )
     }

   }

}
}
