pipeline {
    options {
        ansiColor('xterm')
        buildDiscarder( logRotator(numToKeepStr: '7') )
        disableConcurrentBuilds()
        disableResume()
        skipDefaultCheckout()
    }

    agent { label "le-build" }
    environment {
        version = "${BRANCH_NAME}"
        major = "${BRANCH_NAME == 'master' ? 'master' : BRANCH_NAME.tokenize('+')[0]}"
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

                sh """#!/bin/bash -ex
                    mkdir -p "upload/skin.ConfluenceInuSasha"
                    cp ${zipfile} upload/skin.ConfluenceInuSasha
                    cp -r skin.ConfluenceInuSasha/{addon.xml,resources,changelog.txt} upload/skin.ConfluenceInuSasha
                """
            }
        }

        stage('upload') {
            when { buildingTag() }
            options {
                lock resource: "Kodi-InuSasha-Repo"
            }
            matrix {
                axes {
                    axis {
                        name 'board'
                        values 'Generic', 'RPi2', 'RPi4'
                    }
                }
                environment {
                    arch="${env.board == 'Generic' ? 'x86_64' : 'arm'}"
                }
                
                stages {
                    stage('Upload') {
                        steps {
                            withCredentials([sshUserPrivateKey(credentialsId: '	inu-at-dl.inusasha.de', keyFileVariable: 'identityFile', passphraseVariable: '', usernameVariable: 'userName')]) {
                                sshPut \
                                    remote: [ 'name': 'inusasha.de', 'host': 'inusasha.de', 'user': username, 'allowAnyHosts': true, 'identityFile': identityFile ],
                                    from: "upload/skin.ConfluenceInuSasha",
                                    into: "${upload_path}/${board}/${arch}"
                            }
                        }
                    }
                }
            }
        }

        stage('Regen Addons.xml') {
            when { buildingTag() }
            options {
                lock resource: "Kodi-InuSasha-Repo"
            }
            steps {
                httpRequest "https://dl.inusasha.de/kodi/addons/addons_xml_generator.php"
            }
        }
    }
}
