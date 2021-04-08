def withDockerNetwork(Closure inner) {
  try {
    networkId = UUID.randomUUID().toString()
    sh "docker network create ${networkId}"
    inner.call(networkId)
  } finally {
    sh "docker network rm ${networkId}"
  }
}
node {
  def commit_id
  environment {
      MYSQL_HOST = 'mysql'
      MYSQL_USER = 'root'
      MYSQL_DATABASE = "dbtest"
      MYSQL_ALLOW_EMPTY_PASSWORD = yes
  }
  stage('Preparation') {
    checkout scm
    sh "git rev-parse --short HEAD > .git/commit-id"
    commit_id = readFile('.git/commit-id').trim()
  }
  stage('test') {
    def myTestContainer = docker.image('node:4.6')
    myTestContainer.pull()
    myTestContainer.inside {
      sh 'npm install --only=dev'
      sh 'npm test'
    }
  }
  stage('test with a DB') {
    def mysql = docker.image('mysql')
    mysql.pull()
    def myTestContainer = docker.image('node:4.6')
    myTestContainer.pull()
    withDockerNetwork{ n ->
        mysql.withRun("--network ${n} --name mysql"){ c ->
          myTestContainer.inside("--network ${n} --name node") { // using linking, mysql will be available at host: mysql, port: 3306
                sh 'npm install --only=dev' 
                sh 'npm test'                     
          }             
        }
    }
  }
  stage('docker build/push/run') {            
    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
      def app = docker.build("facundocristaldo/docker-nodejs:${commit_id}", '.').push()
    }
    sh "CONTAINER_ID= $(docker run facundocristaldo/docker-nodejs:${commit_id})"
    sh "docker exec -ti $CONTAINER_ID nodedocker"         
  }
  stage('Docker execution') {
    steps {
      sh "echo 'a change'"
      sh "CONTAINER_ID= $(docker run facundocristaldo/docker-nodejs:${commit_id}))"
    }
  }
}               
