pipeline {
  agent any
  stages{
    stage('build'){
      steps{
        sh '.mvnw clean package -Dcheckstyle.skip'
      }
    }
    stage('Scan'){
      steps{
        sh 'grype dir:. --scope AllLayers'
      }
    }
  }
}
