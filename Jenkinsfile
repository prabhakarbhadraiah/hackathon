node {
	stage 'Checkout'
		checkout scm

	stage 'Build'
		sh 'mvn clean package'

	stage 'Deploy'
		archive 'ProjectName/bin/Release/**'

}
