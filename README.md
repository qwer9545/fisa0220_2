#FISA 2기 클라우드엔지니어링 240220 실습
미션 *** 

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


