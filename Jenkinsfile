pipeline {
    agent { label 'centos7-docker-4c-2g' }
    options {
        timestamps()
        disableConcurrentBuilds()
        preserveStashes()
        quietPeriod(5) // wait a few seconds before starting to aggregate multiple commits into a single build
        durabilityHint 'PERFORMANCE_OPTIMIZED'
    }
    triggers {
        issueCommentTrigger('.*^recheck$.*')
    }
    stages {
        stage('Build Docs') {
            agent {
                dockerfile { 
                    filename 'Dockerfile.docs'
                    reuseNode true
                    args '-u 0:0 --privileged --entrypoint='
                }
            }
            steps {
                sh 'mkdocs build'

                // stash the site contents generated from mkdocs build
                stash name: 'site-contents', includes: 'docs/**', useDefaultExcludes: false
            }
        }

        // back onto the main centos agent (not in docker container)
        stage('Publish to GitHub pages') {
            when {
                beforeAgent true
                expression { edgex.isReleaseStream() }
            }
            agent {
                label 'centos7-docker-4c-2g'
            }
            steps {
                script {
                    edgeXGHPagesPublish(repoUrl: 'git@github.com:edgexfoundry/edgex-docs.git')
                }
            }
        }
    }

    post {
        always {
            edgeXInfraPublish()
        }
        cleanup {
            cleanWs()
        }
    }
}