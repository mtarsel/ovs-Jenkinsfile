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
	sh '''ip -all netns del || echo No namespaces.
	ovs-vsctl del-br br-bond || echo del br-bond
	ovs-vsctl del-br br0 || echo No bridgez
	'''
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

		#ovs-vsctl add-port br0 p1 tag=100
		#ovs-vsctl add-port br0 p2 tag=200
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
/*
#Both ping and netperf are run
#under these 4 combinations of offload settings:
#  gro on gso on tso on tx on
#  gro off gso on tso on tx on
#  gro on gso off tso off tx on
#  gro on gso on tso off tx off
    stage('gro on gso on tso on tx on') {
	sh '''
	if ethtool -k p1 | grep -q "tcp-segmentation-offload: on"; then
		echo "tso on"
		ethtool -K p1 tso off
	else 
		echo "tso off"
	fi
	'''
    }*/


    stage('Setup Bonding') {
        sh '''
	
	echo "resetting ns1 and ns2 for bonding"
	for i in 1 2; do
		#last octet in IP is 10,20
		ip=$(($i*10))

		ip add del 10.0.0.$ip/16 dev p$i
		#only add ip to tap interface in ns
		#ip add add 10.0.0.15/16 dev p$i
		ip netns exec ns$i ip add del 10.0.0.$ip/16 dev tap$i
 		ip netns exec ns$i ip add add 10.0.0.15/16 dev tap$i
		ip link set dev p$i up
		ip netns exec ns$i ip link set dev tap$i up
	done
	'''
	/*
	#TODO remove IP addres so routes are not created
	ip add del 10.0.0.30/16 dev p3
	ip add del 10.0.0.40/16 dev p4
	*/

	/*#install_netcat if needed
	if [ $(rpm -qa | grep ncat | wc -l) -eq 0 ]; then
		yum install nmap-ncat
	fi*/

    }

    stage ('Removing p1 and p2 from br0') {
    sh '''
	ovs-vsctl del-port br0 p1
	ovs-vsctl del-port br0 p2
	'''
    }



    stage('Create br-bond Bridge and Add IP') {
    sh '''
	ovs-vsctl add-br br-bond
	ovs-vsctl add-bond br-bond bond0 p1 p2 bond_mode=active-backup
	ip add add 10.0.0.1/16 dev br-bond
    '''
    }
    
    stage('Ping Active Port and Bring Down') {
        sh '''
	echo "getting active port"
	str=$(ovs-appctl bond/show bond0 | awk ' /p{1,2}/ {print $4 }')
	active="${str:$((${#str}-2)):1}"

	#ping -c2 10.0.0.15
	ping 10.0.0.15 > bond_ping.results 2>&1 &

        echo "bringin down interface p$active..."
        ip netns exec ns$active ip link set tap$active down
        ip link set p$active down
	'''
    }

    stage('Ping New Active Port') {
	sh '''
        #sleep 120
        ping -c2 10.0.0.15 || echo Ping new active
	'''
    }

    stage ('Remove All Bridges'){
	sh '''
	ovs-vsctl del-br br0
	ovs-vsctl del-br br-bond
	'''
    }


   /* stage('Create 1000 Ports') {
        sh '''
	for i in `seq 1 4`; do
		#single-ns $i
		ip link add dum$i type dummy	
		#testitPASS ovs-vsctl add-port $bridge_name p$i
		ovs-vsctl add-port $bridge_name dum$i
		echo "added dum$i"
	done
	'''
    }

	stage('Delete 1000 Ports') {
        sh '''
	for i in `seq 1 4`; do
		#testitPASS ovs-vsctl add-port $bridge_name p$i
		ovs-vsctl del-port $bridge_name dum$i
		ip link del dum$i
	done
	'''
    }

 post {
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
