node {
    stage('Checkout') {
        checkout scm
    }
    stage('Preparation') {
        // remove the previous deployment; "|| true" = don't fail when nothing is there yet
        sh 'docker rm -f todoapp todoappdb || true'
        sh 'docker network create todo-net || true'
    }
    stage('Database') {
        sh '''
            docker run -d --name todoappdb --network todo-net \
              -e MARIADB_ROOT_PASSWORD=sekrit -e MARIADB_DATABASE=todo_db \
              -e MARIADB_USER=todo_usr -e MARIADB_PASSWORD=letmeinplz \
              mariadb:11
            until docker exec todoappdb healthcheck.sh --connect --innodb_initialized; do sleep 3; done
            docker exec -i todoappdb mariadb -utodo_usr -pletmeinplz todo_db < TodoApp/schema.sql
        '''
    }
    stage('Build') {
        sh 'docker build -t todoapp ./TodoApp'
    }
    stage('Deploy') {
        sh '''
            docker run -d --name todoapp --network todo-net -p 8081:8080 \
              -e "ConnectionStrings__TodoDb=Server=todoappdb;Port=3306;Database=todo_db;User=todo_usr;Password=letmeinplz;" \
              todoapp
        '''
    }
    stage('Smoke test') {
        sh 'sleep 5; curl -sf http://172.16.0.10:8081/ > /dev/null && echo "App is up"'
    }
}