pipeline {
    options {
        ansiColor('xterm')
        buildDiscarder( logRotator(numToKeepStr: '7') )
        disableConcurrentBuilds()
        disableResume()
        skipDefaultCheckout()
    }

    agent any
    environment {
        version = "${BRANCH_NAME}"
        major = "9.0"
        name = "skin.ConfluenceInuSasha-${version}"
        zipfile = "${name}.zip"
        upload_path = "public_html/dl.inusasha.de/kodi/addons/${major}"
    }
    
    stages {
        stage('checkout') {
            steps {
                cleanWs()
                dir('skin.ConfluenceInuSasha') {
                    checkout scm
                }
            }
        }
        
        stage('build') {
            steps {
                sh """#!/bin/bash -ex
                    cd skin.ConfluenceInuSasha
                    sed "s|%version%|${version}|" -i addon.xml
                """
                zip zipFile: zipfile,
                    glob: '**/*.*'
            }
        }
        
        stage('upload') {
            when { buildingTag() }
            options {
                lock resource: "Kodi-InuSasha-Repo"
            }
            steps {
                sh """#!/bin/bash -ex
                    mkdir -p "upload/skin.ConfluenceInuSasha"
                    cp ${zipfile} upload/skin.ConfluenceInuSasha
                    cp -r skin.ConfluenceInuSasha/{addon.xml,resources,changelog.txt} upload/skin.ConfluenceInuSasha
                """
                withCredentials([sshUserPrivateKey(credentialsId: '	inu-at-dl.inusasha.de', keyFileVariable: 'identityFile', passphraseVariable: '', usernameVariable: 'userName')]) {
                    sshPut \
                        remote: [ 'name': 'inusasha.de', 'host': 'inusasha.de', 'user': username, 'allowAnyHosts': true, 'identityFile': identityFile ],
                        from: "upload/skin.ConfluenceInuSasha",
                        into: "${upload_path}/RPi2/arm"
                    sshPut \
                        remote: [ 'name': 'inusasha.de', 'host': 'inusasha.de', 'user': username, 'allowAnyHosts': true, 'identityFile': identityFile ],
                        from: "upload/skin.ConfluenceInuSasha",
                        into: "${upload_path}/Generic/x86_64"
                }
                httpRequest "https://dl.inusasha.de/kodi/addons/addons_xml_generator.php"
            }
        }
    }
}
