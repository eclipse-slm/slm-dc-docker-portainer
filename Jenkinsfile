def scenarios = [
    "ubuntu2204": [
        ["setup", "install"],
        ["use", "deploy"],
        ["use", "update"],
        ["use", "undeploy"],
        ["setup", "uninstall"]
    ],
    "ubuntu2004": [
        ["setup", "install"],
        ["use", "deploy"],
        ["use", "update"],
        ["use", "undeploy"],
        ["setup", "uninstall"]
    ],
     "ubuntu1804": [
         ["setup", "install"],
         ["use", "deploy"],
         ["use", "update"],
         ["use", "undeploy"],
         ["setup", "uninstall"]
     ],
    "centos8": [
       ["setup", "install"],
       ["use", "deploy"],
       ["use", "update"],
       ["use", "undeploy"],
       ["setup", "uninstall"]
    ],
     "centos7": [
        ["setup", "install"],
        ["use", "deploy"],
        ["use", "update"],
        ["use", "undeploy"],
        ["setup", "uninstall"]
     ],
    "win10-ssh": [
        ["setup", "install"],
        ["use", "deploy"],
        ["use", "undeploy"],
        ["setup", "uninstall"]
    ]//,
//    "win10-winrm": [
//        ["use", "deploy"],
//        ["use", "undeploy"],
//    ]
]

parallel_stages = [:]

for (kv in mapToList(scenarios)) {
    def platform = kv[0]
    def testList = kv[1]

    parallel_stages[platform] = {

        docker.image('fabos4ai/molecule:4.0.4').inside('-u root') {

            stage("Install dependencies") {
                sh "ansible-galaxy install -f -r requirements.yml"
                sh "ansible-galaxy install -f -r roles/requirements.yml"
            }

            stage("${platform} - Create") {
                sh "cd ./roles/setup && molecule create -s install-${platform}"
            }

            for(int i = 0; i < testList.size(); i++) {
                def role = testList[i][0]
                def scenario = testList[i][1]
                stage("${platform} - ${scenario}") {
                    sh "cd ./roles/${role} && molecule test -s ${scenario} -p ${platform} --destroy never"
                }
            }
        }
    }
}

node {
    checkout scm

    withCredentials([usernamePassword(
        credentialsId: 'jenkins_infra_account',
        usernameVariable: 'VSPHERE_USER',
        passwordVariable: 'VSPHERE_PASSWORD'
    )]) {

        try {
            parallel(parallel_stages)
        } finally {
            stage("Destroy") {
                sh "cd ./roles/setup && molecule destroy -s install"
            }
        }
    }
}

@NonCPS
List<List<?>> mapToList(Map map) {
  return map.collect { it ->
    [it.key, it.value]
  }
}