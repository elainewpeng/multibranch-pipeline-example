pipeline {
    agent any
    triggers {
        cron(env.BRANCH_NAME == 'release' ? 'H(0-30) 15 * * 5' : '')
    }
    parameters {
         string(name: 'DEPLOY_VERSION', defaultValue: '', description: 'Version to deploy')
         choice(
            choices: ["dev","staging", "mvp"],
            description: 'Select environment to deploy',
            name: 'DEPLOY_ENV'
         )
    }
    environment {
        DEV_BRANCH_PATTERN = "^dev-.*"
        GIT_ACCESS_TOKEN = credentials("GIT_ACCESS_TOKEN")
        GIT_URL = "https://${GIT_ACCESS_TOKEN}@github.com/elainewpeng/multibranch-pipeline-example.git"
        MAVEN_REPO_RELEASE = "file://${HOME}/maven-repo/releases"
        MAVEN_REPO_SNAPSHOT = "file://${HOME}/maven-repo/snapshots"

        IS_BUILD_BY_USER = is_build_cause('UserIdCause')
        IS_BUILD_BY_TIMER = is_build_cause('TimerTriggerCause')
        IS_RELEASE_TRIGGER = "${env.IS_BUILD_BY_USER || IS_BUILD_BY_TIMER}"
        
        RUN_RELEASE = "${(env.BRANCH_NAME == 'release') && (IS_RELEASE_TRIGGER == 'true')}"
        RUN_BUILD_FOR_DEPLOY_ONLY = "${}"
        //FixME
        RUN_DEPLOY = "${(env.BRANCH_NAME == 'release') && (IS_RELEASE_TRIGGER == 'true')}"
        DEPLOY_DESC = "Deploy ${DEPLOY_TARGET_ENV}"

        DEPLOY_TARGET_ENV = "${env.RUN_RELEASE ? 'staging' : params.DEPLOY_ENV}"
     }
    stages {
        stage('Debug') {
            steps {
                sh 'printenv'
            }
        }
        stage('Build version for deploy') {
            when { environment name: 'RUN_BUILD_FOR_DEPLOY_ONLY', value: "true" }
            steps {
                echo "Checkout and build version for deploy"
            }
        }
        stage('Release') {
            when { environment name: 'RUN_RELEASE', value: "true" }
            steps {
                echo 'Run release'
            }
        }
        stage('Deploy') {
            when { environment name: 'RUN_DEPLOY', value: "true" }
            steps {
                echo "Deploy to ${DEPLOY_TARGET_ENV}"
            }
        }
    }
}

def is_build_cause(String cause_name) {
    for(cause in currentBuild.getBuildCauses()) {
        if (cause['_class'].contains(cause_name)) {
            return true
        }
    }
    return false
}

