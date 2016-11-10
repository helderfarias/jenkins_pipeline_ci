# Jenkins IC

## Setup
```bash
docker volume create --name jenkins

docker run -d --name jenkins \
   --privileged \
   --restart always \
   -p 50000:50000 -p 8080:8080 \
   -v jenkins:/var/jenkins_home \
   -v /var/run/docker.sock:/var/run/docker.sock \
   -v $(which docker):/usr/local/bin/docker \
   jenkinsci/jenkins
```

## Jenkins Pipeline Script
```bash
node('go') {
    stage "build"

    sh "mkdir -p app/src/authz"
    
    dir("app/src/authz") {
        git "https://github.com/helderfarias/authz-server-mock.git"
        def ws = "${env.workspace}"
        def go = docker.image('helderfarias/gobuild')
        go.inside("-e GOPATH=${ws}/app") { 
            sh "govendor build +local"
        }
    }
}

withEnv(["PATH+Docker=${tool 'Docker'}/bin"]) {
   docker.withServer('ip:porta') {
       def maven = docker.image("maven:alpine")

       maven.inside("-v /var/jenkins_home/.m2:/root/.m2") {
           sh "mvn package"
       }

       step([$class: 'ArtifactArchiver', artifacts: '**/target/*.jar', fingerprint: true])
       step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])            
   }
}

docker.image('maven:3.3.3-jdk8').inside('-v ~/.m2/repo:/ m2repo') {
   sh 'mvn -Dmaven.repo.local=/m2repo clean package' 
}
```

https://dzone.com/refcardz/continuous-delivery-with-jenkins-workflow
