// Membuat Jenkins Pipeline dengan Scripted Pipeline
// Mulai dengan mendefinisikan pipeline
node {
    // Menggunakan Docker sebagai agent untuk menjalankan pipeline dengan mengunduh Docker image bernama node:16-buster-slim dan juga menjalankan container dengan port mapping 3000:3000
    docker.image('node:16-buster-slim').inside('-p 3000:3000 --user root') {
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

        // Untuk mendefinisikan sebuah stage (tahapan) 'Deploy' untuk membuat build dan hasil build ke ec2
        stage('Deploy') {
            withCredentials([string(credentialsId: 'ec2-password', variable: 'EC2_PASSWORD')]) {

            // Mendefinisikasin environtment variables 
            def EC2_HOST = '54.151.249.134'
            def EC2_USER = 'ec2-user'  
            def NGINX_PATH = '/usr/share/nginx/html'

            // Menjalankan script deliver.sh yang berada di direktori jenkins/scripts dari root pada react-app repository
            sh './jenkins/scripts/deliver.sh' 

            // Mengecek build sudah dibuat
            sh 'ls -la build'

            // Menginstal shhpass dan openssh-client
            sh '''
                apt-get update
                apt-get install -y sshpass openssh-client
                which sshpass
            '''

            // Membuat file arsip terkompresi yang berisi seluruh konten dari direktori build.
            sh "tar -czf deploy.tar.gz -C build ."

            // Mengirim file arsip hasil build ke EC2
            sh """
                sshpass -p '${EC2_PASSWORD}' scp -o StrictHostKeyChecking=no deploy.tar.gz ${EC2_USER}@${EC2_HOST}:~/
            """

            // Deploy ke Nginx dengan sshpass
            sh """
                sshpass -p '${EC2_PASSWORD}' ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                    sudo systemctl stop nginx &&
                    sudo rm -rf ${NGINX_PATH}/* &&
                    sudo mkdir -p ${NGINX_PATH} &&
                    sudo tar -xzf ~/deploy.tar.gz -C ${NGINX_PATH} &&
                    sudo chown -R nginx:nginx ${NGINX_PATH} &&
                    sudo systemctl start nginx &&
                    rm ~/deploy.tar.gz
                '
            """
            // Menjalankan perintah sleep untuk menjeda eskekusi pipeline agar aplikasi bisa tetap berjalan selama 1 menit.
            sleep time: 1, unit: 'MINUTES'


            // Menjalankan script kill.sh yang berada di direktori jenkins/scripts dari root pada react-app repository
            sh './jenkins/scripts/kill.sh' 

            // Aplikasi sudah tersedia dan dapat di cek
            echo "Deployment selesai, aplikasi tersedia di http://${EC2_HOST}"
            }
        }
    }
}