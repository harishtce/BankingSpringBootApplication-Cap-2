node{
    
    def tag, dockerHubUser, containerName, httpPort = ""
    
    stage('Prepare Environment'){
        echo 'Initialize Environment'
        tag="3.0"
	mavenHome = tool name: 'mavenHome' , type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
	withCredentials([usernamePassword(credentialsId: 'Dockerhub', usernameVariable: 'dockerUser', passwordVariable: 'dockerPassword')]) {
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
        sh "${mavenCMD} clean package"        
    }
    
    stage('Docker Image Build'){
        echo 'Creating Docker image'
        sh "docker build -t $dockerHubUser/$containerName:$tag --pull --no-cache ."
    }  
	
    stage('Publishing Image to DockerHub'){
        echo 'Pushing the docker image to DockerHub'
        withCredentials([usernamePassword(credentialsId: 'Dockerhub', usernameVariable: 'dockerUser', passwordVariable: 'dockerPassword')]) {
		sh "docker login -u $dockerUser -p $dockerPassword"
		sh "docker push $dockerUser/$containerName:$tag"
		echo "Image push complete"
        } 
    }

     stage('Deleting the containers in local'){
        echo 'Deleting the containers in local'
        sh "docker image rm $dockerHubUser/$containerName"
	echo "Deletion of Containers is complete" 
    }
	
	stage('Ansible Playbook Execution'){
		sh "ansible-playbook -i inventory.yaml kubernetesDeploy.yaml -e httpPort=$httpPort -e containerName=$containerName -e dockerImageTag=$dockerHubUser/$containerName:$tag"
	}
}


