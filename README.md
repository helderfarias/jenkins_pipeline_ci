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
```
