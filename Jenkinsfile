node{
    stage('Git Checkout'){
		git credentialsId: 'github', 
		    url: 'https://github.com/jaymezon/java-app',
			branch: "validator-backend"
	}
	
    stage('Maven Build'){
		sh 'mvn clean package'
	}
}
