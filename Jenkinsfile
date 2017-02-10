#!/usr/bin/env groovy
properties([[$class: 'jenkins.model.BuildDiscarderProperty',
            strategy: [$class: 'LogRotator',
                        numToKeepStr: '5']]])

class Globals {

    static String dockerRunMavenClean =
            "docker run --rm -i " +
                    "-v `pwd`:/workdir -w /workdir " +
                    "-v /var/run/docker.sock:/var/run/docker.sock " +
                    "dl2.homeawaycorp.com/ha-docker/minijava "
}


node {
    wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'XTerm', 'defaultFg': 1, 'defaultBg': 2]) {

        stage('checkout') {
            checkout scm
        }

        stage('Build and Unit Test') {
            echo "Building with gradle"
            sh "${Globals.dockerRunMavenClean} ./gradlew -Prelease.version=1.5.6 install"
            echo `pwd`
            sh "cd exhibitor-standalone/src/main/resources/buildscripts/standalone/gradle/"
            sh "${Globals.dockerRunMavenClean} ../../../../../../../gradlew shadowJar"
        }

        stage('Publish to artifactory') {
            def server = Artifactory.server 'Artifactory_Sungard'
            def uploadSpec = """{
              "files": [
                {
                  "pattern": "exhibitor-standalone/src/main/resources/buildscripts/standalone/gradle/build/libs/exhibitor*.jar",
                  "target": "war3z/exhibitor/"
                }
             ]
            }"""
            server.upload(uploadSpec)
        }
    }
}
