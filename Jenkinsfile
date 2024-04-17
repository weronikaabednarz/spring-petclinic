pipeline 
{
    agent any

    stages {
        stage('Collect') 
        {
            steps 
            {
                git branch: "main", url: "https://github.com/weronikaabednarz/spring-petclinic"
                sh 'git config user.email "weronikaabednarz@gmail.com"'
                sh 'git config user.name "weronikaabednarz"'
            }
        }

        stage('Build') 
        {
            steps 
            {
                script 
                {
                    try 
                    {
                        docker.build("spring-builder", "-f dockerfile_builder .")
                    } 
                    catch (Exception e) 
                    {
                        currentBuild.result = 'FAILURE'
                        error "Błąd w trakcie budowania obrazu Docker: ${e.message}"
                    }
                }
            }
        }

        stage('Test') 
        {
            steps 
            {
                script 
                {
                    try 
                    {
                        docker.build("spring-tester", "-f dockerfile_tester .")
                    } 
                    catch (Exception e) 
                    {
                        currentBuild.result = 'FAILURE'
                        error "Błąd w trakcie testowania obrazu Docker: ${e.message}"
                    }
                }
            }
        }

        stage('Deploy & Publish') 
        {
            agent any
            steps 
            {
                script 
                {
                    try 
                    {
                        def TIMESTAMP = sh(script: 'date +%Y%m%d%H%M%S', returnStdout: true).trim()
                        env.TIMESTAMP = TIMESTAMP
                        def builder_container = docker.image("spring-builder").run("-d --name builder${TIMESTAMP}")
                        def tester_container = docker.image("spring-tester").run("-d --name tester${TIMESTAMP}")
                        sh "docker logs ${builder_container.id} > builder_log.txt"
                        sh "docker logs ${tester_container.id} > tester_log.txt"
                        builder_container.stop()
                        tester_container.stop()
                        sh "tar -czf Artifact_${TIMESTAMP}.tar.gz builder_log.txt tester_log.txt"
                        archiveArtifacts artifacts: "Artifact_${TIMESTAMP}.tar.gz", onlyIfSuccessful: true
                    } 
                    catch (Exception e) 
                    {
                        currentBuild.result = 'FAILURE'
                        error "Błąd w trakcie deployu: ${e.message}"
                    }
                }
            }
        }
    }
}
