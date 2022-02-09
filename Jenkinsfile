properties([
    parameters(
        [
            string(
                name: "ParamString01",
                defaultValue: 'DefaultValue01', 
                description: 'SomeTestParameter'
            ),
            choice(
                name: 'TestChoiceParam01', 
                choices: ['one', 'two', 'three'], 
                description: 'This is my test choise param')
        ]
    )
])

String branchName = env.BRANCH_NAME
String gitCredentials = "github"
String repoUrl = "https://github.com/mikhail-kolganov-clearscale/CSAWSCERT-269_jenkins_groovy.git"

podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    metadata:
        labels: 
            some-label: some-label-value
    spec:
        containers:
            -   name: jnlp
                workingDir: /home/jenkins
                
            -   name: kaniko
                workingDir: /home/jenkins
                image: gcr.io/kaniko-project/executor:debug
                imagePullPolicy: IfNotPresent
                command:
                - /busybox/cat
                tty: true
                volumeMounts:
                -   name: jenkins-docker-cfg
                    mountPath: /kaniko/.docker

        volumes:
        -   name: jenkins-docker-cfg
            projected:
                sources:
                -   secret:
                        name: docker-credentials
                        items:
                            -   key: .dockerconfigjson
                                path: config.json
    ''') {
    node(POD_LABEL) {

        stage('Clone the Git repo') {

            // sh 'printenv | sort'
            // echo $GIT_BRANCH
            // echo $GIT_URL
            git branch: $GIT_BRANCH, credentialsId: $GIT_BRANCH, url: $GIT_URL
            ${WORKSPACE}/mvnw package
            }

        stage ("Build Dokcer Image in Kaniko") {
            container('kaniko', shell: '/busybox/sh') {
                sh  '''#!/busybox/sh
                    /kaniko/executor --context `pwd` --verbosity debug --destination m2hadmin/test-pet-clinic:latest
                    '''
                }
            }

        }

    }

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