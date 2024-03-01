node{


    def tag, dockerHubUser, containerName, httpPort = ""
    
    stage('Prepare Environment'){
        echo 'Initialize Environment'
        tag="3.0"
	withCredentials([usernamePassword(credentialsId: 'DOCKER_CRED', usernameVariable: 'dockerUser', passwordVariable: 'dockerPassword')]) {
		dockerHubUser="$dockerUser"
        }
	containerName="bankingapp"
	httpPort="8989"


    }
    
    stage('Code Checkout'){
        try{
            checkout scm
        }
        catch(Exception e){
            echo 'Exception occured in Git Code Checkout Stage'
            currentBuild.result = "FAILURE"
        }
    }
    
    stage('Maven Build'){
        sh "mvn clean install"        
    }
    
    stage('Docker Image Build'){
        echo 'Creating Docker image'
        sh "docker build -t $dockerHubUser/$containerName:$tag --pull --no-cache ."
    }  
	
    stage('Publishing Image to DockerHub'){
        echo 'Pushing the docker image to DockerHub'
        withCredentials([usernamePassword(credentialsId: 'DOCKER_CRED', usernameVariable: 'dockerUser', passwordVariable: 'dockerPassword')]) {
		sh "docker login -u $dockerUser -p $dockerPassword"
		sh "docker push $dockerUser/$containerName:$tag"
		echo "Image push complete"
        } 
    }    
	
	stage('Ansible Ad-hoc Execution'){
	        sh "ansible localhost -m ping"		
	}
	
	stage('Ansible Playbook Execution'){
		ansiblePlaybook become: true, credentialsId: 'ssh-agent', disableHostKeyChecking: true, installation: 'Ansible', inventory: '/var/lib/jenkins/workspace/orbit-bank/inventory2.yaml', playbook: '/var/lib/jenkins/workspace/orbit-bank/kubernetesDeploy.yaml', vaultTmpPath: ''
	}
}


