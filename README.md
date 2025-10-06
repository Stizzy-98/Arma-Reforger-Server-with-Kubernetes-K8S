# Arma-Reforger-Server-with-Kubernetes-K8S-

Here is my repo for running a Arma Reforger server within Kubernetes! I made this over a weekend after reading that Bohemia had Docker containers that could run the servers. 

Within my homelab I am running a Talos OS cluster https://www.talos.dev/ running Kubernetes (K8S) and Calico for networking to host multiple applications among them an Arma Reforger server ran fully in Kubernetes. The storage backplane on my server is being run with Longhorn https://longhorn.io/ also hosted with Kubernetes. All files posted are free of use and encouraged to be used to learn Kubernetes yourself and host an Arma Reforger server yourself!

I would keep the namespace set at reforger just for easy reference but you can do whatever of course.

**Break Down**

Reforger-storage.yaml is for setting up the storage plane so that it persists between deleting and redeploying the server. Keep in mind it was set up with longhorn so you will have to make changes if you use something else like Rook-CEPH. The reforger-server.yaml is for deploying the actual server itself. When you make a copy of the files go in and edit in your configurations for things such as IP and server preference for name and player count. The sniffer.yaml is for troubleshooting networking when you get the server up. When I was troubleshooting NAT issues it was very helpful in seeing connections coming in to verify that anything was happening. 

**Docker Compose** 

Theoretically this will work in Docker however you will need to go in and change all the volumes to what you want them to be. 

**Networking**

You will need to port forward port 2001 and 17777 both for UDP and set up NAT in your router. I used Pure NAT in my PFSense router for example set the NAT rules and then set the actual interface rules on WAN allowing the traffic to the WAN and the actual IP of the server. If there are networking issues just assume it is NAT. When I got the server up it would be visible in the Reforger browser but it was not joinable from my end because NAT was all messed up and would just loop trying to connect so keep that in mind if you have issues yourself.  

**Useful kubectl commands** 

Monitor the progression of the server install: 

kubectl -n reforger logs -l app=arma-reforger -c install-arma -f

Monitor the server logs:

kubectl -n reforger logs -l app=arma-reforger -c reforger -f

Monitor A2S logs: 

kubectl -n reforger exec deploy/arma-reforger -c reforger -- sh -lc ' 
grep -R --line-number -E "Server registered|Direct Join Code|\[A2S\]" /home/profile/logs || true'
