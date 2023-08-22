pipeline {
    agent any
    triggers {
        cron(env.BRANCH_NAME == 'release' ? 'H(0-30) 15 * * 5' : '')
    }
    parameters {
         string(name: 'DEPLOY_VERSION', defaultValue: '', description: 'Version to deploy')
    }
    environment {
        GIT_ACCESS_TOKEN = credentials("GIT_ACCESS_TOKEN")
        GIT_URL = "https://${GIT_ACCESS_TOKEN}@github.com/elainewpeng/multibranch-pipeline-example.git"
        MAVEN_REPO_RELEASE = "file://${HOME}/maven-repo/releases"
        MAVEN_REPO_SNAPSHOT = "file://${HOME}/maven-repo/snapshots"

        TRIGGER_BY_USER = is_build_cause('UserIdCause')
        TRIGGER_BY_TIMER = is_build_cause('TimerTriggerCause')
        TRIGGER_BY_COMMIT = is_build_cause('BranchEventCause')
        TRIGGER_BY_INDEX = is_build_cause('BranchIndexingCause')

        IS_SCM_BRANCH_TRIGGER = "${TRIGGER_BY_COMMIT || TRIGGER_BY_INDEX}".toBoolean()
        IS_RELEASE_TRIGGER = "${(TRIGGER_BY_USER || TRIGGER_BY_TIMER) &&  !IS_SCM_BRANCH_TRIGGER}".toBoolean()
        IS_DEPLOY_TRIGGER = "${TRIGGER_BY_USER  &&  !IS_SCM_BRANCH_TRIGGER}".toBoolean()

        IS_DEPLOY_BRANCH = "${BRANCH_NAME in ['deploy_mvp']}".toBoolean()
        IS_RELEASE_BRANCH = "${BRANCH_NAME in ['release']}".toBoolean()

        //Variables used by stages
        RUN_RELEASE = "${IS_RELEASE_BRANCH  && IS_RELEASE_TRIGGER}".toBoolean()
        SKIP_BUILD = "${RUN_RELEASE || TRIGGER_BY_INDEX}".toBoolean()

        RUN_DEPLOY = "${IS_DEPLOY_TRIGGER}".toBoolean()
        RUN_DEPLOY_ONLY =  "${IS_DEPLOY_BRANCH  && IS_DEPLOY_TRIGGER}".toBoolean()
        DEPLOY_TARGET_ENV = "${RUN_RELEASE ? 'staging' : (IS_DEPLOY_BRANCH ? 'mvp':'dev')}"
     }
    stages {
        stage('Debug') {
            steps {
                sh 'printenv'
            }
        }
        stage('Checkout Deploy Version') {
            when { environment name: 'RUN_DEPLOY_ONLY', value: "true" }
            steps {
                echo "Checkout Version For Deploy"
            }
        }
        stage('Build') {
            when { environment name: 'SKIP_BUILD', value: "false" }
            steps {
                echo "Build"
            }
        }
        stage('Release') {
            when { environment name: 'RUN_RELEASE', value: "true" }
            steps {
                echo 'Running release'
            }
        }
        stage('Deploy') {
            when { environment name: 'RUN_DEPLOY', value: "true" }
            steps {
                echo "Deploying to ${DEPLOY_TARGET_ENV}"
            }
        }
    }
}

def is_build_cause(String cause_name) {
    for(cause in currentBuild.getBuildCauses()) {
        print(cause['_class'])
        if (cause['_class'].contains(cause_name)) {
            return true
        }
    }
    return false
}

