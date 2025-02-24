        // Membuat Jenkins Pipeline dengan Scripted Pipeline
        // Mulai dengan mendefinisikan pipeline
        node {
            environment {
            EC2_HOST = '54.151.249.134'
            EC2_USER = 'ec2-user'  // atau ec2-user untuk Amazon Linux
            SSH_KEY_ID = 'ec2-ssh-key'
            DEPLOY_PATH = '/var/www/react-app'
            }
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
                stage('Check SSH') {
                    // Memeriksa apakah ssh ada di PATH
                    sh 'echo $PATH'
                    // sh 'ls -l /usr/bin/ssh'  // Memeriksa izin
                    // sh 'cat /usr/bin/ssh'    // Memeriksa apakah file ssh ada
                    sh 'which ssh || echo "SSH not found"'

                    sh 'find / -name ssh 2>/dev/null || echo "SSH not found in any location"'
                }
                stage('Deploy') { 
                    // Menjalankan script deliver.sh yang berada di direktori jenkins/scripts dari root pada react-app repository
                    // sh './jenkins/scripts/deliver.sh' 

                    // Deploy dan jalankan temporary di EC2
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                sh 'apt-get install -y sshpass openssh-client' 

                sh '''
                    ssh -i \$SSH_KEY \$EC2_USER@\$EC2_HOST '
                        sudo systemctl start nginx
                    '
                '''
                    // Menjalankan perintah sleep untuk menjeda eskekusi pipeline agar aplikasi bisa tetap berjalan selama 1 menit.
                    sleep time: 1, unit: 'MINUTES'


                    // Menjalankan script kill.sh yang berada di direktori jenkins/scripts dari root pada react-app repository
                    // sh './jenkins/scripts/kill.sh' 

                    sh """
                        ssh -i \$SSH_KEY \$EC2_USER@\$EC2_HOST '
                            sudo systemctl stop nginx
                        '
                    """

                    
                }
            }
        }
    }