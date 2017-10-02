/*

Written by Mick Tarsel

*/

node ('124') {

    stage('GetVersion') {
        sh "uname -r"
        sh "ovs-vsctl show"
    }

    stage('Verify OVS is running') {
	sh "systemctl is-active openvswitch"
    }

    stage('Clean workspace') {
	sh '''ip -all netns del || echo No namespaces.'''
	sh '''ovs-vsctl del-br br0 || echo No bridges'''
    }

    stage('Create NameSpaces') {

        sh '''
	NUMNS=4
	for i in `seq 1 $NUMNS`; do
		#create name spaces
		ip netns add ns$i

		# create a port pair. the tap$i exists in the ns, p$i on host 
		ip link add tap$i type veth peer name p$i
	
		#attach namespace dev tap$i to ns1 such that tap$i exists only in ns1 
		ip link set tap$i netns ns$i

		#turn on port on OVS. exists on host network
		ip link set dev p$i up

		#add ten to last octet of network ip addr
		IP=$(echo $IP | awk -F. '{printf "%d.%d.%d.%d", 10,$2,$3,$4+10}')
        
		#assign ip addresses.
		ip netns exec ns$i ip add add $IP/16 dev tap$i

		#bring the interface in ns1 up
		ip netns exec ns$i ip link set dev tap$i up

		#add IP address to ovs port
		 ip add add $IP/16 dev p$i
	done
	'''
    }
    stage('Create Bridges') {
       sh '''
	ovs-vsctl add-br br0
	'''
    }
    stage('Attach All NS to Bridge') {
        sh '''
	NS=$(ip netns list | wc -l)

	#attach namespace port to OVS	
	for i in `seq 1 $NS`; do
		ovs-vsctl add-port br0 p$i
	done
	'''
    }
    stage('Setup VLAN') {
        sh '''
	for i in `seq 1 4`; do
		#split tags into 100 and 200 from 4 ns
		tag=$(((($i%2)+1)*100))
	
		ovs-vsctl set port p$i tag=$tag

		#ovs-vsctl add-port br-01 p1 tag=100
		#ovs-vsctl add-port br-01 p2 tag=200
		#p3 tag=100
		#p4 tag=200
	done
	'''
    }
    stage('Test VLAN') {
        sh '''
	#tag=100
	ip netns exec ns1 ping -c2 10.0.0.30
	ip netns exec ns3 ping -c2 10.0.0.10
	
	#tag=200
	ip netns exec ns2 ping -c2 10.0.0.40
	ip netns exec ns4 ping -c2 10.0.0.20

	'''
    }

 /*post {
        always {
            sh '''
		for i in 1 2 3 4; do
			ip link del p$i
		done
		ip -all netns del
		ovs-vsctl del-br br0
		'''
        }
    }*/
}
