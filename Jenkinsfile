node('mac_for_ios') {
  scmVars = checkout scm
  tree = "develop"
  try {
    grpc = "protoc-gen-objcgrpc"
    withEnv(['IROHA_PATH=iroha',
            'SCHEMA_PATH=Schema',
            'PROTO_GEN=ProtoGen',
            'LANG=en_US.UTF-8']) {
      if (scmVars.GIT_LOCAL_BRANCH == 'develop' || scmVars.CHANGE_BRANCH_LOCAL == 'develop' ) {
        stage('prepare') {
          sh(script: "git clone -b master --depth=1 https://github.com/hyperledger/iroha")
          sh(script: "mkdir \$SCHEMA_PATH")
          sh(script: "cp -R \$IROHA_PATH/shared_model/schema \$SCHEMA_PATH/proto")
          sh(script: "rm -rf \$PROTO_GEN && mkdir \$PROTO_GEN")
        }
        stage('build') {
          sh(script: "protoc --plugin=protoc-gen-grpc=\$(command -v ${grpc}) --objc_out=\$PROTO_GEN-iroha --grpc_out=\$PROTO_GEN-iroha --proto_path=./\$SCHEMA_PATH/proto ./\$SCHEMA_PATH/proto/*.proto")
        }
      }
      stage('test') {
        sh(script: "pod lib lint --verbose")
      }
      checkTag = sh(script: 'git describe --tags --exact-match ${GIT_COMMIT}', returnStatus: true)
      if (scmVars.GIT_LOCAL_BRANCH ==~ /(master)/ && checkTag == 0) {
        stage('release') {
          sh(script: "pod trunk push")
        }
      }
    }
  } // end try
  catch(Exception e) {
    currentBuild.result = 'FAILURE'
  } // end catch
  finally {
    cleanWs()
  }
} //end node


