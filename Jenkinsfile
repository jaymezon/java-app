node{
    stage('Git Checkout'){
		git credentialsId: 'github', 
		    url: 'https://github.com/jaymezon/java-app',
			branch: "my-app"
	}
	
    stage('Maven Build'){
		sh 'mvn clean package'
	}
}
