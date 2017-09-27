/*

Written by Mick Tarsel

*/


node ('124') {

    stage('Build') {
        echo 'Building..'
        sh "uname -r"
    }
    stage('Test') {
        echo 'Testing..'
        sh "ovs-vsctl show"
    }
    stage('Deploy') {
        echo 'Deploying....'
        sh "echo 'mick was here'"
    }
}
