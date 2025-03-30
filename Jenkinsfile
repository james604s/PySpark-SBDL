pipeline {
    agent any

    environment {
        PYTHON_VERSION = '3.10.15'
        VENV_NAME = 'venv'
    }

    stages {
        stage('Setup Python Environment') {
            steps {
                sh '''
                    # init pyenv
                    export PYENV_ROOT="$HOME/.pyenv"
                    export PATH="$PYENV_ROOT/bin:$PATH"
                    eval "$(pyenv init -)"
                    eval "$(pyenv virtualenv-init -)"

                    # install specify Python 
                    pyenv install -s $PYTHON_VERSION

                    # build virtualenv（如果還沒建立）
                    pyenv virtualenv --force $PYTHON_VERSION $VENV_NAME

                    # activate virtualenv
                    pyenv activate $VENV_NAME

                    # requirements
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    export PYENV_ROOT="$HOME/.pyenv"
                    export PATH="$PYENV_ROOT/bin:$PATH"
                    eval "$(pyenv init -)"
                    eval "$(pyenv virtualenv-init -)"
                    pyenv activate $VENV_NAME

                    pytest
                '''
            }
        }

        stage('Package') {
            when {
                anyOf { branch "master"; branch "release" }
            }
            steps {
                sh 'zip -r sbdl.zip lib'
            }
        }

        stage('Release') {
            when {
                branch 'release'
            }
            steps {
                sh '''
                    scp -i /home/prashant/cred/edge-node_key.pem -o "StrictHostKeyChecking no" -r \
                    sbdl.zip log4j.properties sbdl_main.py sbdl_submit.sh conf \
                    prashant@40.117.123.105:/home/prashant/sbdl-qa
                '''
            }
        }

        stage('Deploy') {
            when {
                branch 'master'
            }
            steps {
                sh '''
                    scp -i /home/prashant/cred/edge-node_key.pem -o "StrictHostKeyChecking no" -r \
                    sbdl.zip log4j.properties sbdl_main.py sbdl_submit.sh conf \
                    prashant@40.117.123.105:/home/prashant/sbdl-prod
                '''
            }
        }
    }
}
