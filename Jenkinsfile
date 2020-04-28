pipeline {
	agent any
		environment{
			dockerHome=  tool 'myDocker'
			dockerMaven= tool 'myMaven'
			PATH = "$dockerHome/bin:$mavenHome/bin:$path"
		}
	stages{
	stage('Build') {
		steps{	
		sh 'mvn --version'
		sh 'docker version'
		echo "Build" 
		}
	}
	stage('Test') {
		steps{	echo "Test"}
	}
	stage('Intergration Test'){
	 steps{	echo('Integration Test')}
	}
	}	
	post
	{
		success{
			echo "Buils was successful"
		}
		always{
			echo "finished somehow"
		}
	}
}
