/*

Written by Mick Tarsel

*/


node ('124') {

    stage('GetVersion') {
        sh "uname -r"
        sh "ovs-vsctl show"
    }
    stage('VerifyOVS') {
	sh "systemctl is-active openvswitch"
    }
    stage('ClearIt') {
	sh "ip -all netns del"
    }

    stage('CreateNS') {

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
		IP=$(echo $IP | awk -F. '{printf "%d.%d.%d.%d", $1,$2,$3,$4+10}')
        
		#assign ip addresses.
		ip netns exec ns$i ip add add $IP/16 dev tap$i

		#bring the interface in ns1 up
		ip netns exec ns$i ip link set dev tap$i up

		#add IP address to ovs port
		 ip add add $IP/16 dev p$i
	done
	'''
    }
    stage('CreateBridges') {
       sh '''
	ovs-vsctl add-br br0
	'''
    }
    stage('AddPorts') {
        echo 'Deploying....'
        sh "echo 'mick was here'"
    }
    stage('AttachAll') {
        sh '''
	NS=$(ip netns list | wc -l)

	#TODO NS is used here and never checked. NUMNS is not local so it found it anyways
	# maybe [ -z NS] or somethin
	if [ $NS -ne $NUMNS ]; then
		echo "namespaces not setup correctly"
		echo "we can set bash variables?"
	fi

	#attach namespace port to OVS	
	for i in `seq 1 $NS`; do
		ovs-vsctl add-port br0 p$i
	done
	'''
    }
    stage('SetupVLAN') {
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
    stage('TestVLAN') {
        sh '''
	#tag=100
	ip netns exec ns1 ping -c2 10.0.0.30
	ip netns exec ns3 ping -c2 10.0.0.10
	
	#tag=200
	ip netns exec ns2 ping -c2 10.0.0.40
	ip netns exec ns4 ping -c2 10.0.0.20

	'''
    }
    stage('ClearAll') {
        sh '''
	for i in 1 2 3 4; do
		ip link del p$i
	done
	ip -all netns del
	ovs-vsctl del-br br0
	'''
    }


}
