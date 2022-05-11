# Kubeadm

## Worker nodes, update strategy
	1. All at once
	2. One at the time
	3. Add a new Node for overflow, migrate each node

## Upgrade summary
	1. Upgrade kubeadm tool
	2. Upgrade master
	3. Upgrade worker
	4. Upgrade kubelet 

## Upgrade execution
	1) Upgrade kubeadm tool
		a. apt update
		apt install kubeadm=1.20.0-00
			i. apt upgrade -y kubeadm =1.12.0-00
	2) Upgrade master
		a. Kubeadm upgrade plan
		b. kubectl drain <node-to-drain> --ignore-daemonsets
		c. Kubeadm upgrade apply v1.12.0
		d. Upgrade the kubelet if exist in the master
			i. apt upgrade -y kubelet=1.12.0-00
			ii. systemctl restart kubelet
	3) Upgrade worker
		a. ssh node-1
		b. sudo kubeadm upgrade node
        apt update
		c. apt install kubeadm=1.20.0-00
		d. kubectl drain <node-to-drain> --ignore-daemonsets
		e. kubeadm upgrade node
		f. Upgrade the kubelet if exist in the master
			i. apt upgrade -y kubelet=1.12.0-00
			ii. systemctl daemon-reload
			iii. systemctl restart kubelet
            kubectl uncordon node01