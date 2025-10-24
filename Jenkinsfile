pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        S3_BUCKET = 'study-edge-pydah1'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Setup AWS CLI') {
            steps {
                sh '''
                    echo "🔧 Checking and installing AWS CLI if missing..."
                    if ! command -v aws >/dev/null 2>&1; then
                        if command -v apt-get >/dev/null 2>&1; then
                            sudo apt-get update -y
                            sudo apt-get install -y awscli
                        elif command -v yum >/dev/null 2>&1; then
                            sudo yum install -y awscli
                        else
                            echo "No package manager found — installing via pip..."
                            python3 -m pip install --user awscli
                            export PATH="$HOME/.local/bin:$PATH"
                        fi
                    else
                        echo "✅ AWS CLI already installed."
                    fi
                    aws --version
                '''
            }
        }

        stage('Upload to S3') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'aws-creds',
                    usernameVariable: 'AWS_ACCESS_KEY_ID',
                    passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                )]) {
                    sh '''
                        echo "🚀 Uploading files to S3 bucket: ${S3_BUCKET}"
                        export AWS_DEFAULT_REGION=${AWS_REGION}

                        # Upload project files (ignore .git & Jenkinsfile)
                        aws s3 sync . s3://${S3_BUCKET}/ \
                            --exclude ".git/*" \
                            --exclude "Jenkinsfile" \
                            --acl private

                        echo "✅ Upload completed successfully!"
                    '''
                }
            }
        }
    }
}
