pipeline{
    agent any
    stages{
        stage("BBBBBBB"){
            steps{
                echo "========executing BBBBBB========"
                sh  'env'
            }
            // post{
            //     always{
            //         echo "========always========"
            //     }
            //     success{
            //         echo "========A executed successfully========"
            //     }
            //     failure{
            //         echo "========A execution failed========"
            //     }
            // }
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