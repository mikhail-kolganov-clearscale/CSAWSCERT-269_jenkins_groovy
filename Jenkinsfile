
properties([
    [$class: 'RebuildSettings', autoRebuild: false, rebuildDisabled: false],
    [$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', artifactDaysToKeepStr: '3', artifactNumToKeepStr: '4', daysToKeepStr: '3', numToKeepStr: '5']],
    parameters(
        [
            booleanParam(name: 'BuildTrigger', defaultValue: true, description: 'Do we need to build the App?'),
            booleanParam(name: 'PushTrigger', defaultValue: true, description: 'Do we need to push the build image to the registry?'),
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
                value: 'RUN ALL TESTS,TEST GROUP 1,TEST GROUP 2,-------------,Test1-1,Test1-2,Test2-1,Test2-2,Test2-3, -------------, SKIP ALL TESTS ',
                visibleItemCount: 12)
        ]
    )
])

TEST_GROUPS = [
    'TEST GROUP 1' : ["Test1-1", "Test1-2"],
    'TEST GROUP 2' : ["Test2-1", "Test2-2", "Test2-3"]
]

String branchName = env.BRANCH_NAME
String gitCredentials = "MyGitHub"
String repoUrl = "https://github.com/mikhail-kolganov-clearscale/CSAWSCERT-269_jenkins_groovy.git"



parallelTests = [
    'Test1-2' : [ 'Test1-2__A', 'Test1-2__B', 'Test1-2__C'],
    'Test2-1' : ['Test2-1_A', 'Test2-1_B']
]

def generateStep(String stepName){
        return {
            timeout(15) {
                echo "===================> Executint Test: ${stepName}"
            }
        }
}

def generateStage(String test, String testGroup) {

    if (parallelTests.containskey(test)) {
        return {
            stage("${testGroup} : ${test}") {
                parallelTests.getAtt(test).each {
                    stepName ->
                    generateStep("${test} :: ${stepName}")
                }
            }
        } 
    } 

    return {
        stage("${testGroup} : ${test}") {
            generateStep(test)
        }
    }
}

def generateTestList(String testGroupName) {
    testToExecute = []
    if ( env.TESTS_TO_EXECUTE.contains('RUN ALL TESTS') ) {
        TEST_GROUPS.each { 
            group ->
            testToExecute.addAll(group.value)
        }
    } else {
        selectedTests = env.TESTS_TO_EXECUTE.split(',')
        testToExecute = TEST_GROUPS.findAll { selectedTests.contains(it)}
    }
    return testToExecute
}



podTemplate(yaml: readTrusted('BuildPodTemplate.yaml')) {
      node(POD_LABEL) { // gets a pod with both docker and maven
        stage('Clone the Repo') {
            sh 'printenv | sort'
            git branch: branchName, credentialsId: gitCredentials, url: repoUrl
            sh 'pwd && ls -la'
            }

        if ( env.BuildTrigger.toBoolean() ){
            stage('Build the App') {
                echo '======= BUILDING ========'
                sh '${WORKSPACE}/mvnw package'
            }
            if ( env.PushTrigger.toBoolean() ) {
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

        if ( env.TestTrigger.toBoolean() ){

            TEST_GROUPS.each {
                testGroupName, testStages ->

                listOfTestsToExecute = generateTestList(testGroupName)

                if(!listOfTestsToExecute.isEmpty()) {
                    stage("Test: testGroupName") {
                        parallel listOfTestsToExecute.collectEntries {
                            test ->
                            ["${test}": generateStage(test, testGroupName)]
                        }
                    }
                }
            }

        }
    }
}

