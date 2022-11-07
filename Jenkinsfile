pipeline 
{
    options 
    {
        timestamps()
    }
    agent any
    tools
    {
        maven 'Maven3.6.2' 
        jdk 'jdk8'
    }
    environment 
    {
        
        MAIN = 'FALSE'
        TAG = ''
        VERSION = ''
        PREFIX = ''
        SUFFIX = ''
        TAGER = ''
        BRANCH = ''
    }
    stages 
    {
        stage("Checkout") 
        {
            
            steps 
            {
                script 
                {
                    STAGE = 'Checkout'
                }
//                updateGitlabCommitStatus name: 'Checkout', state: 'pending'
                deleteDir()
                checkout scm
                
                
                withCredentials([string(credentialsId: 'api_token', variable: 'TOKEN')]) { 
                    sh "git fetch http://jenkins:$TOKEN@3.11.200.189/Maciob/suggest-lib-multibranch --tags"
                }
                script
                {
                    BRANCH = env.BRANCH_NAME
                    if ("${BRANCH}".contains("main"))
                    {
                        sh "echo 'TRUE'"
                        MAIN = 'TRUE'
                        TAG = sh(returnStdout: true, script:"git tag | grep main | tail -1 | cut -d '/' -f2").trim()
                        PREFIX = sh(returnStdout: true, script:"echo '${TAG}' | cut -d '.' -f1 ").trim()
                        SUFFIX = sh(returnStdout: true, script:"echo '${TAG}' | cut -d '.' -f2 ").trim()
                        VERSION = "${SUFFIX}" as int
                        VERSION = VERSION + 1
                        TAGER = sh(returnStdout: true, script:"echo 'main/${PREFIX}.${VERSION}-SNAPSHOT'").trim()
                        sh "echo '${TAGER}'"
                    }
                    else
                    {
                        PREFIX = sh(returnStdout: true, script:"echo '${BRANCH}' | cut -d '/' -f2").trim()
                        TAGNAME = sh(returnStdout: true, script:"git tag | grep ${PREFIX} | tail -1").trim()
                        if("${TAGNAME}"=='')
                        {
                            TAGER = "${PREFIX}.0"
                        }
                        else
                        {
                            if("${TAGNAME}".contains("main"))
                            {
                                HEAD = '1' as int
                                TAIL = '2' as int
                                
                                while ("${TAGNAME}".contains("main"))
                                {
                                    TAGNAME = sh(returnStdout: true, script:"git tag | grep ${PREFIX} | tail -${TAIL} | head -${HEAD}").trim()
                                    HEAD = HEAD + 1
                                    TAIL = TAIL + 1
                                }
                            }
                            
                            PREFIX = sh(returnStdout: true, script:"echo '${TAGNAME}' | cut -d '.' -f1-2").trim()
                            SUFFIX = sh(returnStdout: true, script:"echo '${TAGNAME}' | cut -d '.' -f3").trim()
                            VERSION = "${SUFFIX}" as int
                            VERSION = VERSION + 1
                            TAGER = sh(returnStdout: true, script:"echo '${PREFIX}.${VERSION}'").trim()
                            sh "echo '${TAGER}'"
                        }
                    }
                }

//                updateGitlabCommitStatus name: 'Checkout', state: 'success'
            }
        }
        stage('Compile') 
        {
            steps 
            {
//                updateGitlabCommitStatus name: 'Build', state: 'pending'
                script 
                {
                    STAGE = 'Build'
                    sh "mvn compile"
                }
//                updateGitlabCommitStatus name: 'Build', state: 'success'
            }
        }
        stage('Test') 
        {
            steps 
            {
//                updateGitlabCommitStatus name: 'Test', state: 'pending'
                script 
                {
                    STAGE = 'Test'
                    sh "mvn test"
                }
//                updateGitlabCommitStatus name: 'Test', state: 'success'
            }
        }
        stage('Deploy') 
        {
            steps 
            {
//                updateGitlabCommitStatus name: 'Deploy', state: 'pending'

                script 
                {
                    STAGE = 'Deploy'

                    configFileProvider([configFile(fileId: '8412b7f1-a630-4f33-9b49-fe3e1de2ad03',variable: 'MAVEN_SETTINGS_XML')]) 
                    {
                        sh "mvn versions:set -s $MAVEN_SETTINGS_XML -DnewVersion=${TAGER}"
                        sh "mvn deploy -s $MAVEN_SETTINGS_XML"
                    }
                }
//                updateGitlabCommitStatus name: 'Deploy', state: 'success'
            }
        }
        stage('Deploy to GIT')
        {
            when 
            {
                expression { "${MAIN}" == 'FALSE' }
            }
            steps 
            {
                script 
                {
                    sh "echo '${TAGER}'"
                    STAGE = 'Deploy to GIT'
                    try
                    {
                        sh "git checkout ${BRANCH}"
                    }
                    catch(exc)
                    {
                        sh "git checkout -b ${BRANCH}"
                    }
                }
//                updateGitlabCommitStatus name: 'Deploy to GIT', state: 'pending'
                sh 'git config --global user.email "jenkins@jenkins.jenkins"'
                sh 'git config --global user.name "jenkins"'
                sh 'git add .'
                sh "git commit -m '[ci-skip] ${TAGER}'"
                sh "git tag '${TAGER}'"
                withCredentials([string(credentialsId: 'api_token', variable: 'TOKEN')]) { 
                    sh 'git push http://Maciob:$TOKEN@3.11.200.189/Maciob/suggest-lib-multibranch'   
                    sh "git push http://Maciob:$TOKEN@3.11.200.189/Maciob/suggest-lib-multibranch refs/tags/${TAGER}"
                }
//                updateGitlabCommitStatus name: 'Deploy to GIT', state: 'success'
            }
        }
    }
}
