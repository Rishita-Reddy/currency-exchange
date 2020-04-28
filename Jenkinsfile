pipeline {
	agent any
	stages{
	stage('Build') {
		echo "Build"
	}
	stage('Test') {
		echo "Test"
	}
	stage('Intergration Test'){
		echo('Integration Test')
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
