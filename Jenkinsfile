/* TODO 
- Add a config map to kubernetes default or public namesapce which is having all needed values 
1.Registry URL
2.Anchor API end point
- Add a secret for all components
1.Registry 
2.Anchor
3.MySQL
4.MediaWiki
- FIndout a way to do scan before push
*/
def label = "build-push-scan-${UUID.randomUUID().toString()}"
def registry_url = "10.128.0.2:5000"

podTemplate(name: 'docker', cloud: 'Kubernetes', label: label, yaml: """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: docker:1.11
    command: ['cat']
    tty: true
    volumeMounts:
    - name: dockersock
      mountPath: /var/run/docker.sock
  volumes:
  - name: dockersock
    hostPath:
      path: /var/run/docker.sock
"""
) 
{
    node(label) {
        stage('Build') {
                    git 'https://github.com/ansilh/demo-middleware.git'
                        container('docker'){
                            docker.withRegistry('https://' + registry_url,'registry'){
                            def dockerImage = docker.build("middleware:${BUILD_NUMBER}")
                            dockerImage.push()
                        }
                    }
        }
        stage('Scan') {
               sh 'pwd > workspace'
                workspace = readFile('workspace').trim()
                def imageLine =  registry_url + "/middleware:${BUILD_NUMBER} " + workspace + "/Dockerfile"
                writeFile file: 'anchore_images', text: imageLine
                sh 'cat anchore_images'
                anchore name: 'anchore_images'
        }
    }
}