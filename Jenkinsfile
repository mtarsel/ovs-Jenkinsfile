/*

Written by Mick Tarsel

*/


node ('124') {

    stage('GetVersion') {
        sh "uname -r"
        sh "ovs-vsctl show"
    }
    stage('ObtainTest') {
	sh "cd /root/mick; wget https://raw.githubusercontent.com/mtarsel/ovs-testplan/master/runall.sh;"
    }
    stage('RunTestALL') {
        echo 'Testing..'
        sh "chmod +x /root/mick/runall.sh"
	sh "source /root/mick/runall.sh"
    }
    stage('Deploy') {
        echo 'Deploying....'
        sh "echo 'mick was here'"
    }
}
