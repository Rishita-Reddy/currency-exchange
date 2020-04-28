pipeline {
	agent any
	stages{
	stage('Build') {
		steps{	echo "Build" }
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
