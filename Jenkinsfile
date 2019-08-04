import hudson.tasks.test.AbstractTestResultAction
def SERVICE_NAME="netcoreapi"
def SUBDOMAIN="sub9"
def PACKAGE_NAME="${env.BRANCH_NAME}-netcore-api.${SUBDOMAIN}.1.0.${env.BUILD_NUMBER}"

node {
    def remote = [:]

    if (env.BRANCH_NAME == "dev") {
      stage('Cleaning ENV') {
          // bat "IF EXIST Publish RMDIR /S /Q Publish"
          // bat "IF EXIST obj RMDIR /S /Q obj"
          // bat "IF EXIST bin RMDIR /S /Q bin"
          deleteDir()
          dir("${workspace}@tmp") {
                deleteDir()
            }
      }
      stage('Clone repository') {
          /* repository cloned to our workspace */
          checkout scm
      }
      stage('DEV: Restore Packages') {
          /* This restoring of the packages of the application. */
          bat "dotnet restore netcore-api.sln"
          // bat "dotnet restore Api.Test/netcore-api.csproj"
      }
      stage('DEV: Clean') {
          /* This clean the solution. */
          bat "dotnet clean netcore-api.sln"

      }
      stage('DEV: Build') {
          /* This builds the solution */
          // bat "dotnet publish netcore-api.sln -o Publish -c Release -r win10-x86"
          bat "dotnet build api/netcore-api.csproj --configuration Release -o Publish"
          bat "dotnet build Api.Test/Api.Test.csproj --configuration Debug"

      }
      stage('DEV: Unit Test') {
        Printing = bat returnStatus: true, script: "\"dotnet\" test \"${workspace}/netcore-api.sln\" --logger \"trx;LogFileName=unit_tests.xml\" --no-build"
        println("Printing: $Printing")
        // step([$class: 'MSTestPublisher', testResultsFile:"Api.Test/TestResults/unit_tests.xml", failOnError: true, keepLongStdio: true])

        if ( (Printing != 0)  ){
          error("Unit Test FAILED...")
        }
    		else
    		{
    			println("Unit Test PASSED...")

    		}
      }

      stage('DEV: Pack') {
          /* This will create zip */
          zip zipFile: "${PACKAGE_NAME}.zip", archive: false, dir: 'api/Publish'
          bat "dir"

      }

      /* This SSH Session deploys app into AppServer */
      withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'AppServer',
      usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']])
      {
        remote.name = 'appserver'
        remote.host = "${env.ServerIP}"  //From ENVIRONMENT VARIABLE
        remote.user = "${USERNAME}"
        remote.password = "${PASSWORD}"
        remote.allowAnyHosts = true
        if (env.BRANCH_NAME == "dev") {
          stage('DEV: Deploy Artifact') {
            sshPut remote: remote, from: "${PACKAGE_NAME}.zip", into: "deployments/packages/"
            sshCommand remote: remote, command: "ls -al deployments/${env.BRANCH_NAME}/"
          }
          stage('DEV: Run Application') {
            sshCommand remote: remote, command: "rm -rf deployments/${env.BRANCH_NAME}/${SUBDOMAIN}*"
            sshCommand remote: remote, command: "unzip deployments/packages/${PACKAGE_NAME}.zip -d deployments/${env.BRANCH_NAME}/${SUBDOMAIN}/"
            sshCommand remote: remote, command: "./deployments/systemd/deploy.sh ${SUBDOMAIN} ${env.BRANCH_NAME}"
            // for extracting into multiple directory: ${PACKAGE_NAME}
            //     sshGet remote: remote, from: 'abc.sh', into: 'bac.sh', override: true
            //     sshRemove remote: remote, path: "deployments/${env.BRANCH_NAME}", failOnError: false
          }
        }

      }

    }

}
