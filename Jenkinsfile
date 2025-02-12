// Membuat Jenkins Pipeline dengan Scripted Pipeline
// Mulai dengan mendefinisikan pipeline
node {
    // Menggunakan Docker sebagai agent untuk menjalankan pipeline dengan mengunduh Docker image bernama node:16-buster-slim dan juga menjalankan container dengan port mapping 3000:3000
    docker.image('node:16-buster-slim').inside('-p 3000:3000') {
        // Untuk mendefinisikan sebuah stage (tahapan) 'Build' untuk melakukan proses build
        stage('Build') {
            // Menjalankan perintah npm untuk menginstall dependencies yang diperlukan untuk menjalankan aplikasi React App 
            sh 'npm install'
        }

        // Untuk mendefinisikan sebuah stage (tahapan) 'Test' untuk menjalankan tes
        stage('Test') {
            // Menjalankan script test.sh yang berada di direktori jenkins/scripts dari root pada react-app repository.
            sh './jenkins/scripts/test.sh'
        }
    }
}