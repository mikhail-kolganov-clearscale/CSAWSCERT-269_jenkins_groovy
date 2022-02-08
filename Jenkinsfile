pipeline{
    agent any
    stages{
        stage("BUILD"){
            steps{
                echo "========executing ./mvnw spring-boot:build-image ========"
                dir("${WORKSPACE}"){
                    sh  './mvnw spring-boot:build-image'
                }
            }
        }
    }
    post{
        always{
            echo "========always========"
            echo "${WORKSPACE}"
        }
        success{
            echo "========pipeline executed successfully ========"
        }
        failure{
            echo "========pipeline execution failed========"
        }
    }
}