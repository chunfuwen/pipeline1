// blatantly ripped off from https://jenkins.io/doc/pipeline/examples/#parallel-multiple-nodes

def labels = ["rhel-6", "rhel-7", "fedora-29"]
def builders = [:]
for (x in labels) {
  def label = x

  builders[label] = {
    node("swarm && rcm-tools-jslave && ${label}") {
      checkout scm
      stage("Install dependencies on ${label}") {
        if (label.contains('fedora')) {
          sh "sudo dnf -y install python36 python3-detox python{2,3}-coverage python{2,3}-qpid-proton gcc"
        } else {
          echo "All dependencies are present in the image"
        }
      }
      stage("Unit tests on ${label}") {
        if (label.contains('fedora')) {
          sh "detox -v"
          junit "xunit-*.xml"
          step([$class: "CoberturaPublisher", coberturaReportFile: "coverage-*.xml"])
        } else {
          sh """
            nosetests \
              --with-xunit --xunit-file=nosetests-${label}.xml \
              --with-coverage --cover-inclusive --cover-branches \
              --cover-package=rhmsg.activemq,rhmsg.cli \
              --cover-xml --cover-xml-file=coverage-${label}.xml
          """
          junit "nosetests-${label}.xml"
          step([$class: "CoberturaPublisher", coberturaReportFile: "coverage-${label}.xml"])
        }
      }
    }
  }
}

parallel builders
