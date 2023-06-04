node{
    
    stage('Clone repo'){
        git credentialsId: 'GIT-Credentials', url: 'https://github.com/prapri77/maven-web-app.git'
    }
    
    stage('Maven Build'){
        def mavenHome = tool name: "Maven-3.8.5", type: "maven"
        sh "${mavenHome}/bin/mvn package"
       sh 'mv target/01-maven-web-app*.war target/srmweb.war'
    }
    
    stage('SonarQube analysis') {       
        withSonarQubeEnv('Sonar-Server-7.8') {
       	sh "mvn sonar:sonar"    	
    }
        
    stage('upload war to nexus'){
	steps{
		nexusArtifactUploader artifacts: [	
			[
				artifactId: '01-maven-web-app',
				classifier: '',
				file: 'target/01-maven-web-app.war',
				type: war		
			]	
		],
		credentialsId: 'nexus3',
		groupId: 'in.ashokit',
		nexusUrl: '',
		protocol: 'http',
		repository: 'ashokit-release'
		version: '1.0.0'
	}
}
    
    stage('Build Image'){
        sh 'docker build -t prasanth77/mavenwebapp .'
    }
    
    stage('Push Image'){
        withCredentials([string(credentialsId: 'DOCKER-CREDENTIALS', variable: 'DOCKER_CREDENTIALS')]) {
            sh 'docker login -u prasanth77 -p ${DOCKER_CREDENTIALS}'
        }
        sh 'docker push prasanth77/mavenwebapp'
    }
    
    stage('Deploy App'){
        kubernetesDeploy(
            configs: 'maven-web-app-deploy.yml',
            kubeconfigId: 'Kube-Config'
        )
    }  
	    stage('Docker deployment'){
   sh 'docker run -d -p 8090:8080 --name tomcattest prasanth77/mavenwebapp' 
   }
}
