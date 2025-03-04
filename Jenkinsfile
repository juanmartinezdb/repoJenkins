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
                sh "echo 'Copiando el WAR al contenedor de Tomcat'"
                sh '''
                    if [ -f "codigo/webapp/target/webapp.war" ]; then
                        docker cp codigo/webapp/target/webapp.war cep:/usr/local/tomcat/webapps/
                        echo 'WAR copiado con éxito'
                    else
                        echo "ERROR: No se encontró el archivo WAR, no se puede copiar."
                        exit 1
                    fi
                '''
            }
        }
    }
}
