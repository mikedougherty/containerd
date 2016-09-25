wrappedNode(label: "linux && x86_64") {
  stage("pre") {
    deleteDir()
    checkout scm
  }

  def img
  stage("build image") {
    img = docker.build("dockerbuildbot/containerd:${gitCommit()}")
  }
  def returnCode = -1
  stage("run tests") {
    try {
      returnCode = sh(script: "docker run --privileged --rm --name '${env.BUILD_TAG}' ${img.id} make test 2>&1 | tee test-results.txt", returnStatus: true)
    } finally {
      sh "docker rmi -f ${img.id} ||:"
    }
  }

  if (returnCode != 0) {
    currentBuild.result = 'UNSTABLE'
  }

  stage("post") {
    withTool("go2xunit") {
      sh "go2xunit < test-results.txt > test-results.xml"
    }
    archiveArtifacts('test-results.*')
    junit(
      testResults: 'test-results.xml',
      keepLongStdio: true
    )
  }
}
