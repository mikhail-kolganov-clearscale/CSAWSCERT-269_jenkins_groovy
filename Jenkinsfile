
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



podTemplate(yaml: readTrusted('BuildPodTemplate.yaml') {
    podTemplate(containers: [containerTemplate(image: $KanikoImage, name: 'kaniko', ttyEnabled: true, resources: $LimitBuilderBool ? ${} : '{}')]) {
      node(POD_LABEL) { // gets a pod with both docker and maven
        stage('Build the App') {
            git branch: branchName, credentialsId: gitCredentials, url: repoUrl
            sh 'pwd && ls -la'
            // sh '${WORKSPACE}/mvnw package'
            }

        stage ("Build Docker Image in Kaniko") {
            container(name: 'kaniko', shell: '/busybox/sh') {

                sh 'ls -la'

                // sh  '''#!/busybox/sh
                //     /kaniko/executor --context `pwd` --verbosity debug --destination m2hadmin/test-pet-clinic:latest
                //     '''
                }
            }
        }
    }
}



// podTemplate(yaml: InitilaBuildPodTemplateYaml ) {
//     node(POD_LABEL) {

//         stage('Build the App') {
//             git branch: branchName, credentialsId: gitCredentials, url: repoUrl
//             sh '${WORKSPACE}/mvnw package'
//             }

//         stage ("Build Docker Image in Kaniko") {
//             container(name: 'kaniko', shell: '/busybox/sh') {

//                 sh 'ls -la'

//                 // sh  '''#!/busybox/sh
//                 //     /kaniko/executor --context `pwd` --verbosity debug --destination m2hadmin/test-pet-clinic:latest
//                 //     '''
//                 }
//             }

//         }

//     }

// timeout(unit: 'SECONDS', time: 150) {
//     podTemplate(yaml: '''
//         apiVersion: v1
//         kind: Pod
//         metadata:
//             labels: 
//                 some-label: some-label-value
//         spec:
//             containers:
//                 -   name: kaniko
//                     image: gcr.io/kaniko-project/executor:debug
//                     imagePullPolicy: IfNotPresent
//                     command:
//                     - sleep
//                     args:
//                     - 99d
//         ''') {
//         node(POD_LABEL) {

//             stage ("Stage 2 - b") {
//                 container('kaniko') {
//                     echo POD_CONTAINER // displays 'busybox'
//                     sh 'hostname'
//                     unstash name: 'TestStash01'
//                     sh 'cat test_file.txt; ls -la'
//                     sh 'prinenv | sort'
//                 }
//             }

//             }

//         }
// }