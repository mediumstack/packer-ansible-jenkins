#!groovy

node {

  currentBuild.result = "SUCCESS"

  // set the docker command to run each stage
  env.PACKER_CMD = "docker run -v $HOME/.aws:/root/.aws:ro  --rm --network host -w /app -v ${WORKSPACE}:/app hashicorp/packer:light"

  // checkout the git repo
  stage ('Checkout') {
    checkout scm
  }

  // if the parameter is set to validatePacker...
  if (env.desiredAction == 'validatePacker') {

    try {

      // First stage, pull the docker image
      stage ('Pull packer image') {
        ansiColor('xterm') {
          sh 'docker pull hashicorp/packer:light'
        }
      }
  
      // Validate the packer config
      stage ('Packer validate') {
        ansiColor('xterm') {
          sh '${PACKER_CMD} validate jenkins-packer-ec2.json'
        }
      }
  
      // Inspect the packer config
      stage ('Packer inspect') {
        ansiColor('xterm') {
          sh '${PACKER_CMD} inspect jenkins-packer-ec2.json'
        }
      }
  
      // cleanup the workspace after each run
      cleanWs()
    }
    catch (err) {

      currentBuild.result = 'FAILURE'
      throw err
    }
  }

  // if the parameter is set to buildPacker...
  if (env.desiredAction == 'buildPacker') {

    try {

      // First stage, pull the docker image
      stage ('Pull packer image') {
        ansiColor('xterm') {
          sh 'docker pull hashicorp/packer:light'
        }
      }
  
      // Validate the packer config
      stage ('Packer validate') {
        ansiColor('xterm') {
          sh '${PACKER_CMD} validate jenkins-packer-ec2.json'
        }
      }
  
      // Inspect the packer config
      stage ('Packer inspect') {
        ansiColor('xterm') {
          sh '${PACKER_CMD} inspect jenkins-packer-ec2.json'
        }
      }
  
      // Wait for approval
      input 'Build packer image?'
  
      // Build the packer config
      stage ('Packer build') {
        ansiColor('xterm') {
          sh '${PACKER_CMD} build jenkins-packer-ec2.json'
        }
      }
  
      // Collect artifacts for later use
      stage ('Gather artifacts') {
  
        // Get the AMI ID
        AMI_ID = sh(returnStdout: true, script: """grep artifact_id manifest.json  | awk '{print \$2}' |  sed 's/"//g' | sed 's/,//g' |cut -d':' -f2""").trim()
  
        // Get the AMI region
        AMI_REGION = sh(returnStdout: true, script: """grep artifact_id manifest.json  | awk '{print \$2}' |  sed 's/"//g' | sed 's/,//g' |cut -d':' -f1""").trim()
  
        // Get the packer_run_uuid
        PACKER_RUN_UUID = sh(returnStdout: true, script: """grep packer_run_uuid manifest.json  | awk '{print \$2}' |  sed 's/"//g' | sed 's/,//g'""").trim()
  
        // Get the snapshot-id
        SNAP_ID = sh(returnStdout: true, script: """aws ec2 describe-images --filter Name=tag:packer_run_uuid,Values=${PACKER_RUN_UUID} | jq ".Images[0].ImageId,.Images[0].BlockDeviceMappings[0].Ebs.SnapshotId" | sed -n '2 p' | sed 's/"//g'""").trim()
  
        // Create a new file to be stored as an artifact with the relevant data
        sh "echo ami: ${AMI_ID} > AMI_ID.afct"
        sh "echo region: ${AMI_REGION} > REGION.afct"
        sh "echo snapshot-id: ${SNAP_ID} >> SNAPSHOT_ID.afct"
      }
  
      stage ("Archive build output") {
        // Archive the build output artifacts.
        archiveArtifacts artifacts: 'manifest.json'
        archiveArtifacts artifacts: 'AMI_ID.afct'
        archiveArtifacts artifacts: 'REGION.afct'
        archiveArtifacts artifacts: 'SNAPSHOT_ID.afct'
      }
  
      // Clean the workspace
      cleanWs()
    }
    catch (err) {

      currentBuild.result = 'FAILURE'
      throw err
    }

  }

}
