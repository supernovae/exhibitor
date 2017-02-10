#!/usr/bin/env groovy
properties([[$class: 'jenkins.model.BuildDiscarderProperty',
            strategy: [$class: 'LogRotator',
                        numToKeepStr: '5']]])
class Globals {

    static String dockerRunMavenClean =
            "docker run --rm -i " +
                    "-v `pwd`:/workdir -w /workdir " +
                    "-v /var/run/docker.sock:/var/run/docker.sock " +
                    "dl2.homeawaycorp.com/ha-docker/minijava"
}


node {
    wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'XTerm', 'defaultFg': 1, 'defaultBg': 2]) {

        stage('checkout') {
            checkout scm
        }

        stage('Build and Unit Test') {
            echo "Building with gradle"
            sh """
                {Globals.dockerRunMavenClean} ./gradlew -Prelease.version=${EXHIBITOR_VERSION} install
                cd exhibitor-standalone/src/main/resources/buildscripts/standalone/gradle/
                {Globals.dockerRunMavenClean} ../../../../../../../gradlew shadowJar
            """
        }

        stage('Publish to artifactory') {
            def server = Artifactory.server 'my-server-id'
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