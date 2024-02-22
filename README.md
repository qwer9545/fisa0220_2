#FISA 2기 클라우드엔지니어링 실습

240220 미션 

1. github에 fisa240220_2레포지터리 생성
2. step07_citest 프로젝트 생성
    1. get 방식 메소드 구현
3. jenkins에 item 개발
    1. item명: step04review
4. 결론
    1. step03 빌드 결과와 같이
    2. -rwxr-xr-x 1 gradlew 파일이 실행 가능한 권한 부여해 주셔야 함
    3. 참고: gradlew 파일이 윈도우에서 생성시에는 읽기만 가능한 권한으로 생성
5. 학습: gitbash & jenkins Item 생성 & githun 활용 & gradle 주의사항
webhook test 16

------------------ Jenkins 구성
pipeline {
    agent any

    stages {
        stage('git clone') {
            steps {
                echo 'git clone'
                
                git branch: 'main', 
                credentialsId: 'credentialId', 
                url: 'https://github.com/qwer9545/fisa0220_2.git'
                
            }
        }
        
        stage('list view') {
            steps {
                echo 'list start'
                
                sh ''' ls -al '''
                sh ''' pwd '''
            }
        }
        
    }
}


--------------------------------------------------
240222 EC2 instance간 CI(무중단 배포) test
1. EC2 t2.2xlarge(instance 1), t2.micro(instance 2) 인스턴스 생성
2. instance1에서 docker image 기반 jenkins pipeline 구성
3. instance1에서 main branch에서의 git clone[Stage clone]
4. instance1에서 clone 파일 .gradle build[Stage build]
5. instance1에서 2로 ssh연결 후 jar 배포[Stage deploy]
6. script.sh 실행으로 instance2에서의 jar 실행 
7. github webhook 설정으로 수정 발생 시 instance 2에 자동으로 배포 및 jar 실행

------------------------------------------------------------------
jenkinsfile

pipeline {
    agent any
    
    stages {
        stage('clone') {
            steps {
                echo 'git clone'
                
                git branch: 'main', 
                url: 'https://github.com/qwer9545/fisa0220_2.git'
            }
        }
        
        stage('build') {
            steps {
                dir('') {
                    sh'''
                    echo 'build start'
                    ./gradlew clean build
                    '''
                    
                    sh ''' ls -al '''
                    sh ''' pwd '''
                }
            }
        }
        
        stage('deploy') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: 'myec2_102', 
                            transfers: [
                                sshTransfer(
                                    cleanRemote: false, 
                                    excludes: '', 
                                    execCommand: 'sh /home/ubuntu/script.sh', 
                                    execTimeout: 120000, 
                                    flatten: false, 
                                    makeEmptyDirs: false, 
                                    noDefaultExcludes: false, 
                                    patternSeparator: '[, ]+', 
                                    remoteDirectory: '', 
                                    remoteDirectorySDF: false, 
                                    removePrefix: 'build/libs/', 
                                    sourceFiles: 'build/libs/*SNAPSHOT.jar'
                                )
                            ], 
                            usePromotionTimestamp: false, 
                            useWorkspaceInPromotion: false, 
                            verbose: true
                        )
                    ]
                )
            }
        }
    }
}

------------------------------------------------------------------
script.sh(from bongsub)

#!/bin/bash

echo "Start Spring Boot Application!"
CURRENT_PID=$(ps -ef | grep java | grep dokotlin | awk '{print $2}')
echo "$CURRENT_PID"

 if [ -z $CURRENT_PID ]; then
echo ">현재 구동중인 어플리케이션이 없으므로 종료하지 않습니다."

else
echo "> kill -9 $CURRENT_PID"
kill -9 $CURRENT_PID
sleep 10
fi
 echo ">어플리케이션 배포 진행!"
nohup sudo java -jar /home/ubuntu/test1/step06_citest-0.0.1-SNAPSHOT.jar >> /home/ubuntu/test1/logs/javaApp.log &

