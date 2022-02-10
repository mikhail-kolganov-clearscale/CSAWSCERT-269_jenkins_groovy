
properties([
    parameters(
        [
            string(
                name: "KanikoImage",
                defaultValue: 'gcr.io/kaniko-project/executor:debug', 
                description: 'Kaniko Image path'
            ),

            booleanParam(name: 'LimitBuilderBool', defaultValue: true, description: 'Limit the Builder POD resources?'),

            string(
                name: "BuilderCpuLimit",
                defaultValue: '200m', 
                description: 'Set the CPU Limit for the Builder POD'
            ),
            string(
                name: "BuilderMemLimit",
                defaultValue: '500Mi', 
                description: 'Set the CPU Limit for the Builder POD'
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
            memory: ${BuilderMemLimit}
            cpu: ${BuilderCpuLimit}
        limits:
            memory: ${BuilderMemLimit}
            cpu: ${BuilderCpuLimit}
"""



podTemplate(yaml: readTrusted('BuildPodTemplate.yaml')) {
    podTemplate(containers: [containerTemplate(image: $KanikoImage, name: 'kaniko', ttyEnabled: true, resources: $LimitBuilderBool ? ${} : '{}')]) {
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

