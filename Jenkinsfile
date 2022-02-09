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
                - name: jenkins-docker-cfg
                mountPath: /kaniko/.docker
        volumes:
        - name: jenkins-docker-cfg
            projected:
            sources:
            - secret:
                name: docker-credentials
                items:
                    -   key: .dockerconfigjson
                        path: config.json
    ''') {
    node(POD_LABEL) {

        stage ("Build Sping-Boot App") {
            sh './mvnw package'
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