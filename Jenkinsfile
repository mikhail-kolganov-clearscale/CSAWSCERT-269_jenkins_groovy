
properties([
    parameters(
        [
            booleanParam(name: 'BuildTrigger', defaultValue: true, description: 'Do we need to build the App?'),
            booleanParam(name: 'PushImage', defaultValue: true, description: 'Do we need to push the build image to the registry?'),

            string(
                name: "ImagePushDestination",
                defaultValue: 'm2hadmin/test-pet-clinic', 
                description: 'Kaniko Image path'
            )
        ]
    )
])

String branchName = env.BRANCH_NAME
String gitCredentials = "MyGitHub"
String repoUrl = "https://github.com/mikhail-kolganov-clearscale/CSAWSCERT-269_jenkins_groovy.git"


podTemplate(yaml: readTrusted('BuildPodTemplate.yaml')) {
      node(POD_LABEL) { // gets a pod with both docker and maven
        stage('Clone the Repo') {
            sh 'printenv | sort'
            git branch: branchName, credentialsId: gitCredentials, url: repoUrl
            sh 'pwd && ls -la'
            }

        if ( env.BuildTrigger ){
            stage('Build the App') {
                echo '======= BUILDING ========'
                sh '${WORKSPACE}/mvnw package'
            }
            stage ("Build Docker Image in Kaniko") {
                container(name: 'kaniko', shell: '/busybox/sh') {
                    sh  """#!/busybox/sh
                        /kaniko/executor --context `pwd` --verbosity debug --destination ${env.ImagePushDestination}:latest
                        """
                }
            }
        }
    }
}