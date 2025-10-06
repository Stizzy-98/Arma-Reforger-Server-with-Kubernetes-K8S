# Arma-Reforger-Server-with-Kubernetes-K8S

Here is my repo for running an Arma Reforger server within Kubernetes! I made this over a weekend after reading that Bohemia had Docker containers that could run the servers. 

Within my homelab I am running a Talos OS cluster https://www.talos.dev/ running Kubernetes (K8S) and Calico for networking to host multiple applications among them an Arma Reforger server ran fully in Kubernetes. The storage backplane on my server is being run with Longhorn https://longhorn.io/ (my raid card is set to IT mode to expose all drives) also hosted with Kubernetes and deployed using Helm. All files posted are free of use and encouraged to be used to learn Kubernetes and host an Arma Reforger server yourself!

**Break Down**

Reforger-Storage.yaml is for setting up the storage plane so that all your data for the arma server install and mods persists between deleting and redeploying the server. This design is extremely helpful when it comes to testing changes to the config so you dont have to redownload everything everytime you tear it down and bring it back up. Keep in mind it was set up with longhorn so you will have to make changes if you use something else like Rook-CEPH. 

The reforger-server.yaml is for deploying the actual server itself. When you make a copy of the files go in and edit in your configurations for things such as IP, machine hostname, and server preferences for name and player count. Another file the modded file is an example of how the mods need to be formatted in the yaml. You can look for all the components for each mod by going to https://reforger.armaplatform.com/workshop. Alternatively if you have a server that you like to play on go to it in Reforger and look at the mod list using the puzzle piece in the top right of the screen and you will be able to see the json for the mods that the server uses and you can copy them from there.  

The sniffer.yaml is for troubleshooting networking when you get the server up. When I was troubleshooting NAT issues it was very helpful in seeing connections coming in to verify that anything was happening. You will need to port forward port 2001 and 17777 both for UDP and set up NAT rules for both in your router. I used Pure NAT in my PFSense router for example set the NAT rules and then set the actual interface rules on WAN allowing the traffic to the WAN and the actual IP of the server. 

If there are networking issues just assume it is NAT. So long as the server is visible in browser and it doesnt crash when you try and connect to it then it is NAT not the server itself have a buddy try and join outside the network to confirm or hop on a seperate network aside from the router you use to troubleshoot further and confirm. In my own experience when I got the server running it would be visible in the Reforger browser but it was not joinable from my end because NAT wasn't properly configured and my connection would just loop trying to connect so keep that in mind if you have this issue yourself.

**Docker Compose** 

Theoretically this will work in Docker however you will need to go in and combine both yamls and change all the volumes and networking to what you need them to be for a dockerized setup. The server that I'm running the server on has 2 CPUs in it with 10 cores each that run at 3.0ghz and I have 256gb of RAM and the server never ran over 15% CPU and 5.5gb of RAM usage on vanilla reforger. While it was running my friends and I were spawning everything and anything to stress test it seeing if it would crash leveling Monti and maxing out the amount of spawnable AI and using mortars having them go to town on each other fighting. So you can use that for reference to gauge whether your hardware can handle running the server.

**Useful kubectl commands** 

Monitor the progression of the server install: 

kubectl -n reforger logs -l app=arma-reforger -c install-arma -f

Monitor the server logs:

kubectl -n reforger logs -l app=arma-reforger -c reforger -f

Monitor A2S logs: 

kubectl -n reforger exec deploy/arma-reforger -c reforger -- sh -lc ' 
grep -R --line-number -E "Server registered|Direct Join Code|\[A2S\]" /home/profile/logs || true'

Command to run the sniffer to watch for player connections coming to the server:

kubectl -n reforger run sniffer --rm -it \
  --image=corfr/tcpdump \
  --overrides='{"spec":{"hostNetwork":true,"dnsPolicy":"ClusterFirstWithHostNet","nodeSelector":{"kubernetes.io/hostname":"YOURHOSTNAME"}}}' \
  -- tcpdump -n -i any udp port 2001
