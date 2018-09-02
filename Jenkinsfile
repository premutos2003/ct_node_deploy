def userInput = true
def didTimeout = false
node {

    stage("Clone app to workspace")
            {
                deleteDir()
                sh 'git clone ${GIT_URL} app'
                sh 'echo cloning app to workspace'
            }
    stage("Clone infrastructure config to workspace") {
        sh 'git clone https://github.com/premutos2003/ct_node_mongo.git'
    }
    stage("Build Docker image/artifact") {
        sh '''#!/bin/bash -xe
    mv ./ct_node_mongo/Dockerfile ./
    mv ./ct_node_mongo/infrastructure/docker-compose.yml ./
    cd app
    entrypoint=$(jq -r .scripts.start package.json)
if [ "$entrypoint" = "null" ];then
	echo no start script
	entrypoint=$(jq -r .main package.json)
	if [ "$entrypoint" != "null" ];then
		echo $entrypoint main entrypoint
		entrypoint="pm2 start $entrypoint -i max  --no-daemon"
	else
		echo no main
		if [ -e index.js ];then
			entrypoint="pm2 start index.js -i max  --no-daemon"
			echo index.js found $entrypoint
		else
			echo no index.js found
			if [ -e app.js ];then
				entrypoint="pm2 start app.js -i max  --no-daemon"
				echo app.jws found $entrypoint
			else
				echo no app.js found
				if [ -e server.js ];then
					entrypoint="pm2 start server.js -i max  --no-daemon"
					echo server.js found $entrypoint
				else
					echo no entrypoint found
				fi
			fi
		fi

	fi
else
if
echo $entrypoint | grep -q node
then
entrypoint=${entrypoint/node/pm2 start}
entrypoint="$entrypoint -i max --no-daemon"
echo $entrypoint
fi
fi
cd ..
    echo Building docker image...
    docker build -t ${PROJECT_NAME}  --build-arg entry="${entrypoint}" --build-arg port=${APP_PORT}  --build-arg folder=app .
    docker save -o ${PROJECT_NAME}.tar ${PROJECT_NAME}:latest
    gzip ${PROJECT_NAME}.tar
    ls
    mv ${PROJECT_NAME}.tar.gz ./ct_node_mongo/infrastructure
    docker rmi ${PROJECT_NAME}
    aws s3 cp ${PROJECT_NAME}.tar.gz s3://app-state-${STACK}-${PROJECT_NAME}/state/0.${EXECUTOR_NUMBER} --region ${REGION}
    ls
    '''
    }
}