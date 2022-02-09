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
            -   name: busybox
                image: busybox
                command:
                - sleep
                args:
                - 99d
            -   name: alpine
                image: alpine
                command:
                - sleep
                args:
                - 99d
    ''') {
    node(POD_LABEL) {

        stage ("Stage 1 - A") {
            container('busybox') {
                echo POD_CONTAINER // displays 'busybox'
                sh 'hostname'
                sh 'echo "TEST TEXT!!!" > test_file.txt '
            }
        }
        stage ("Stage 2 - B") {
            container('alpine') {
                    echo "------- step 2-1"
                    sh 'cat test_file.txt'
                }
            }

        }

    }

timeout(unit: 'SECONDS', time: 50) {
    podTemplate(yaml: '''
        apiVersion: v1
        kind: Pod
        metadata:
            labels: 
                some-label: some-label-value
        spec:
            containers:
                -   name: busybox2
                    image: busybox
                    command:
                    - sleep
                    args:
                    - 99d
        ''') {
        node(POD_LABEL) {

            stage ("Stage 2 - b") {
                container('busybox2') {
                    echo POD_CONTAINER // displays 'busybox'
                    sh 'hostname'
                    sh 'cat test_file.txt '
                    sh 'prinenv | sort'
                }
            }

            }

        }
}