// Membuat Jenkins Pipeline dengan Scripted Pipeline
// Mulai dengan mendefinisikan pipeline
node {
    // Definisi environment variables di level yang tepat
    def EC2_HOST = '54.151.249.134'
    def EC2_USER = 'ec2-user'  
    def NGINX_PATH = '/usr/share/nginx/html'
    
    // Membungkus dengan withCredentials
    withCredentials([string(credentialsId: 'ec2-password', variable: 'EC2_PASSWORD')]) {
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

            stage('Deploy') { 
                // Menjalankan script deliver.sh yang berada di direktori jenkins/scripts dari root pada react-app repository
                sh './jenkins/scripts/deliver.sh' 
                // sh 'rm -rf /var/lib/dpkg/lock-frontend'
                // sh 'rm -rf /var/lib/dpkg/frontend'
                // sh 'ls -la dist'
                sh 'ls -la build'
                sh 'apt-get update && apt-get install -y sshpass openssh-client' 

                sh "tar -czf deploy.tar.gz -C build ."

                sh "sshpass -p '${EC2_PASSWORD}' scp -o StrictHostKeyChecking=no deploy.tar.gz ${EC2_USER}@${EC2_HOST}:~/"

                            // Deploy ke Nginx
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

                echo "Deployment selesai, aplikasi tersedia di http://${EC2_HOST}"

                
            }
        }
    }
}