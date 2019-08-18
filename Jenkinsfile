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
        major = "9"
        name = "skin.ConfluenceInuSasha-${version}"
        zipfile = "${name}.zip"
        upload_path = "public_html/dl.inusasha.de/libreelec/addons"
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
                    sed "s|%major%|${major}|" -i addon.xml
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
                    mkdir -p "${major}/skin.ConfluenceInuSasha"
                    cp ${zipfile} ${major}/skin.ConfluenceInuSasha
                    cp -r skin.ConfluenceInuSasha/{addon.xml,resources,changelog.txt} ${major}/skin.ConfluenceInuSasha
                """
                withCredentials([sshUserPrivateKey(credentialsId: '	inu-at-dl.inusasha.de', keyFileVariable: 'identityFile', passphraseVariable: '', usernameVariable: 'userName')]) {
                    sshPut \
                        remote: [ 'name': 'inusasha.de', 'host': 'inusasha.de', 'user': username, 'allowAnyHosts': true, 'identityFile': identityFile ],
                        from: major,
                        into: upload_path
                }
                httpRequest "https://dl.inusasha.de/libreelec/addons/addons_xml_generator.php"
            }
        }
    }
}
