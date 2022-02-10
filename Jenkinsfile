properties([
    // [$class: 'RebuildSettings', autoRebuild: false, rebuildDisabled: false],
    [$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', artifactDaysToKeepStr: '3', artifactNumToKeepStr: '4', daysToKeepStr: '3', numToKeepStr: '5']],
    
    parameters(
        [
            booleanParam(name: 'BuildTrigger', defaultValue: false, description: 'Do we need to build the App?'),
            booleanParam(name: 'PushTrigger', defaultValue: false, description: 'Do we need to push the build image to the registry?'),
            booleanParam(name: 'TestTrigger', defaultValue: true, description: 'Do we need to run tests?'),
            string(
                name: "ImagePushDestination",
                defaultValue: 'm2hadmin/test-pet-clinic', 
                description: 'Kaniko Image path'
            ),
            extendedChoice(
                defaultValue: 'RUN ALL TESTS',
                description: 'Multi select list of stages to be executed during this execution',
                multiSelectDelimiter: ',', 
                name: 'TESTS_TO_EXECUTE',
                quoteValue: false, 
                saveJSONParameterToFile: false,
                type: 'PT_MULTI_SELECT',
                value: 'RUN ALL TESTS,-------------,TEST_GROUP_1,TEST_GROUP_2,TEST_GROUP_3,-------------,Test1-1,Test1-2,Test2-1,Test2-2,Test2-3,Test3-1,Test3-2,Test3-3',
                visibleItemCount: 12)
        ]
    )
])

TEST_GROUPS = [
    'TEST_GROUP_1' : ["Test1-1", "Test1-2"],
    'TEST_GROUP_2' : ["Test2-1", "Test2-2", "Test2-3"],
    'TEST_GROUP_3' : ["Test3-1", "Test3-2", "Test3-3"]
]

String branchName = env.BRANCH_NAME
String gitCredentials = "MyGitHub"
String repoUrl = "https://github.com/mikhail-kolganov-clearscale/CSAWSCERT-269_jenkins_groovy.git"



// just an empy map to fill with actial Tests we want to execute
TESTS = [:]


def generateStage(String test, String testGroup) {

    echo "Generating the Stage: ${test}"
    return {
        stage("TestGroup: ${testGroup} Test: ${test}") {
            return catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                dir(path: "${WORKSPACE}/my_tests/${testGroup}/") {
                    echo "--- Execuring the test: ${testGroup}/${test}"
                    sh "cat ./${test}.txt"
                }
            }
        }
    }
}


def generateTestList(String testName) {
    testToExecute = [:] // create an empty Map
    if ( testName == 'RUN ALL TESTS') {   // if we have 'RUN ALL TESTS' selected, just return all our TEST_GROUPS amp
        testToExecute = TEST_GROUPS
    } else if (TEST_GROUPS.containsKey(testName)) {   // else, if we have a Group Name - put all this group into map
            testToExecute.put(testName, TEST_GROUPS[testName])
            return testToExecute
    }

    TEST_GROUPS.each {   // for individual tests, put them indivudually
        groupKey, groupVal ->
        if ( groupVal.contains(testName) )
            testToExecute=["${groupKey}": [testName]]         
    }
    return testToExecute

}


podTemplate(yaml: readTrusted('BuildPodTemplate.yaml')) {
    node(POD_LABEL) { 
        stage('Clone the Repo') {
            // sh 'printenv | sort'
            git branch: branchName, credentialsId: gitCredentials, url: repoUrl
            sh 'pwd && ls -la'
            }

        stage ('OWASP Dependency-Check Vulnerabilities') {
            container(name: 'maven') {
                dir(path: "${WORKSPACE}/complete/") {
                    echo '======= CHECK DEPENDECIES ========'
                    sh '''
                        pwd
                        ls -la
                        which mvn
                    '''
                    sh  "mvn dependency-check:check"
                    
                    dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
                }
            }
        }

        stage('SonarQube analysis'){
            container(name: 'maven') {
                dir(path: "${WORKSPACE}/complete/") {
            withSonarQubeEnv(credentialsId: 'SonarToken', installationName: 'Sonar-Server') {
                sh 'mvn sonar:sonar -Dsonar.dependencyCheck.jsonReportPath=target/dependency-check-report.json -Dsonar.dependencyCheck.xmlReportPath=target/dependency-check-report.xml -Dsonar.dependencyCheck.htmlReportPath=target/dependency-check-report.html'
            }}
        }}

        if ( env.BuildTrigger.toString().toBoolean() ){
            stage('Build the App'){ 
                container(name: 'maven') {
                    dir(path: "${WORKSPACE}/complete/") {
                        echo '======= BUILDING ========'
                        sh 'mvn package'
                    }
                }
            }


            if ( env.PushTrigger.toString().toBoolean() ) {
                stage ("Build Docker Image in Kaniko") {
                    container(name: 'kaniko', shell: '/busybox/sh') {
                        sh  """#!/busybox/sh
                            /kaniko/executor --context `pwd` --verbosity debug --destination ${env.ImagePushDestination}:latest
                            """
                    }
                }  
            } else {
                echo "===== SKIPPING IMAGE PUSH due to env.PushTrigger: ${env.PushTrigger} ===="
            }
            } else {
                    echo "===== SKIPPING THE BUILD due to env.BuildTrigger: ${env.BuildTrigger} ===="
            }
        
    }
}

podTemplate(yaml: readTrusted('TestPodTemplate.yaml')) {
    node(POD_LABEL) {
        if ( env.TestTrigger.toString().toBoolean()){
            timeout(15){
            container(name: 'testpod') {
                stage('Clone the Repo') {
                        git branch: branchName, credentialsId: gitCredentials, url: repoUrl
                    }
                // parse input parameter and find those tests we need to execute
                selectedTests = env.TESTS_TO_EXECUTE.split(',')
                stage('TESTING') {
                
                    if(selectedTests.size() > 0 ) {
                        selectedTests.each {
                            test ->
                            generateTestList(test).each {
                            key, val ->
                            String group = key.toString()
                            if( TESTS.containsKey(group) ){
                                TESTS[group].addAll(val as Set)
                            } else {
                                TESTS.put(group, val as Set)
                            }
                            }
                        }
                    }

                    TESTS.each {
                        testGroup, testSet ->
                        stage(testGroup) {
                            parallel testSet.collectEntries {
                                ["${it}": generateStage(it, testGroup)]
                            }
                            }
                        }
                    }
                }
            }
        }
    }
}

