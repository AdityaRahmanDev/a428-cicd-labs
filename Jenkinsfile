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
            // Menjalankan script test.sh yang berada di direktori jenkins/scripts dari root pada react-app repository
            sh './jenkins/scripts/test.sh'
        }
        // Untuk mendefinisikan sebuah stage (tahapan) 'Manual Approval' untuk pengguna bisa memilih apakah klik Proceed (melanjutkan eksekusi pipeline ke tahap Deploy) atau Abort (menghentikan eksekusi pipeline)
        stage('Manual Approval') {
            // Menlankan perintah input message untuk lanjut ke tahap deploy atau menghentikan eksekusi
            input message: 'Lanjutkan ke tahap Deploy?' 
        }
        stage('Deploy') { 
            // Menjalankan script deliver.sh yang berada di direktori jenkins/scripts dari root pada react-app repository
            sh './jenkins/scripts/deliver.sh' 
            // Menjalankan perintah sleep untuk menjeda eskekusi pipeline agar aplikasi bisa tetap berjalan selama 1 menit.
            sleep time: 1, unit: 'MINUTES'
            // Menjalankan script kill.sh yang berada di direktori jenkins/scripts dari root pada react-app repository
            sh './jenkins/scripts/kill.sh' 
        }
    }
}
