// // Membuat Jenkins Pipeline dengan Scripted Pipeline
// // Mulai dengan mendefinisikan pipeline
// node {
//     // Menggunakan Docker sebagai agent untuk menjalankan pipeline dengan mengunduh Docker image bernama node:16-buster-slim dan juga menjalankan container dengan port mapping 3000:3000
//     docker.image('node:16-buster-slim').inside('-p 3000:3000') {
//         // Untuk mendefinisikan sebuah stage (tahapan) 'Build' untuk melakukan proses build
//         stage('Build') {
//             // Menjalankan perintah npm untuk menginstall dependencies yang diperlukan untuk menjalankan aplikasi React App 
//             sh 'npm install'
//         }

//         // Untuk mendefinisikan sebuah stage (tahapan) 'Test' untuk menjalankan tes
//         stage('Test') {
//             // Menjalankan script test.sh yang berada di direktori jenkins/scripts dari root pada react-app repository.
//             sh './jenkins/scripts/test.sh'
//         }
//     }
// }


pipeline {
    agent {
        docker {
            image 'node:16-buster-slim'
            args '-p 3000:3000'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                sh './jenkins/scripts/test.sh'
            }
        }

        stage('Manual Approval') {
            steps {
                input message: 'Lanjutkan ke tahap Deploy?' 
            }
        }
        stage('Deploy') { 
            steps {
                sh './jenkins/scripts/deliver.sh' 
                // input message: 'Sudah selesai menggunakan React App? (Klik "Proceed" untuk mengakhiri)' 
                sleep time: 1, unit: 'MINUTES'
                sh './jenkins/scripts/kill.sh' 
            }
        }
    }
}