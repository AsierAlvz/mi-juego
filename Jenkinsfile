pipeline {
    agent any

    environment {
        FIREBASE_TOKEN = credentials('firebase-token')
    }

    stages {
        stage('Instalacion de dependencias') {
            steps {
                sh 'npm install'
            }
        }

        stage('Analisis de codigo Lint') {
            steps {
                sh 'npm run lint'
            }
        }

        stage('Ejecucion de tests') {
            steps {
                sh 'npm run test:unit -- --run'
            }
        }

        stage('Build del proyecto') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Deploy en Firebase') {
    steps {
        sh '''printf '{\n  "hosting": {\n    "public": "dist",\n    "ignore": ["firebase.json","**/.*","**/node_modules/**"],\n    "rewrites": [{"source": "**","destination": "/index.html"}]\n  }\n}' > firebase.json'''
        sh '''printf '{\n  "projects": {\n    "default": "mi-juego-d636c"\n  }\n}' > .firebaserc'''
        sh 'firebase deploy --token $FIREBASE_TOKEN --non-interactive'
    }
}
    }

    post {
        success {
            echo 'Pipeline completado. App desplegada en Firebase.'
        }
        failure {
            echo 'El pipeline ha fallado. Revisa los logs.'
        }
    }
}