pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                dir('codigo') {
                    // Clonar el código fuente
                    git 'https://github.com/juanmartinezdb/maven-project.git'

                    // Ejecutar Maven (usando withMaven para mejor integración)
                    withMaven(maven: 'Maven 3') {
                        sh "mvn clean package -DskipTests"
                    }
                }
            }
            post {
                success {
                    archiveArtifacts artifacts: 'codigo/webapp/target/*.war', fingerprint: true
                }
            }
        }

        stage('Test') {
            steps {
                sh "echo 'Realización de algunos Test'"    
            }
        }

        stage('Container') {
            steps {
                sh "echo 'Preparando imagen de Docker'"
                dir('contenedor') {
                    withCredentials([usernamePassword(credentialsId: 'gitprueba', passwordVariable: 'password', usernameVariable: 'usuario')]) {
                        git 'https://github.com/juanmartinezdb/dockerimage-from-jenkins-pipelin.git'
                        sh '''
                            # Verificar si el WAR existe antes de copiarlo
                            if [ -f "../codigo/webapp/target/webapp.war" ]; then
                                cp ../codigo/webapp/target/webapp.war .
                                git add .
                                git config --local user.email 'testif@iesalixar.org'
                                git config --local user.name 'testif@iesalixar.org'
                                git commit -m 'Desde Jenkinsfile'
                                git config --local credential.helper "!f() { echo username=\\$usuario; echo password=\\$password; }; f"
                                git push origin master
                            else
                                echo "ERROR: No se encontró el archivo WAR"
                                exit 1
                            fi
                        '''
                    }
                }
            }
        }

        stage('Deploy') {
    steps {
        sh "echo 'Desplegando'"
        // Verifica o crea el directorio en el contenedor Tomcat
        sh "docker exec cep mkdir -p /usr/local/tomcat/webapps/"
        // Copia el WAR desde Jenkins al contenedor Tomcat (nota: esto se hace desde el host)
        sh "docker cp serverJenkins:/var/jenkins_home/workspace/Pipeline-Jenkins/codigo/webapp/target/webapp.war cep:/usr/local/tomcat/webapps/"
        sh "echo 'WAR copiado con éxito'"
    }
}

        }
    }
}
