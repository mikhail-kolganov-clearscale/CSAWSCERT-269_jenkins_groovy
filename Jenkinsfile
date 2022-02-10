
properties([
    parameters(
        [
            string(
                name: "KanikoImage",
                defaultValue: 'gcr.io/kaniko-project/executor:debug', 
                description: 'Kaniko Image path'
            ),

            choice(
                name: 'TestChoiceParam01', 
                choices: ['one', 'two', 'three'], 
                description: 'This is my test choise param')
        ]
    )
])

String branchName = env.BRANCH_NAME
String gitCredentials = "MyGitHub"
String repoUrl = "https://github.com/mikhail-kolganov-clearscale/CSAWSCERT-269_jenkins_groovy.git"

InitPodResources = """
        requests:
            memory: ${env.BuilderMemLimit}
            cpu: ${env.BuilderCpuLimit}
        limits:
            memory: ${env.BuilderMemLimit}
            cpu: ${env.BuilderCpuLimit}
"""



podTemplate(yaml: readTrusted('BuildPodTemplate.yaml')) {
    podTemplate(containers: [containerTemplate(image: env.KanikoImage, name: 'kaniko', ttyEnabled: true, command: 'cat')]) {
      node(POD_LABEL) { // gets a pod with both docker and maven
        stage('Build the App') {
            git branch: branchName, credentialsId: gitCredentials, url: repoUrl
            sh 'pwd && ls -la'
            }

        stage ("Build Docker Image in Kaniko") {
            container(name: 'kaniko', shell: '/busybox/sh') {

                sh 'ls -la'
                }
            }
        }
    }
}