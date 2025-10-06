# Arma-Reforger-Server-with-Kubernetes-K8S-

Here is my repo for running a Arma Reforger server within Kubernetes! I made this over a weekend after reading that Bohemia had Docker containers that could run the servers. 

Within my homelab I am running a Talos OS cluster https://www.talos.dev/ running Kubernetes (K8S) to host multiple applications among them an Arma Reforger server ran fully in Kubernetes. The storage backplane on my server is being run with Longhorn https://longhorn.io/ also hosted with Kubernetes. All files posted are free of use and encouraged to be used to learn Kubernetes yourself and host an Arma Reforger server yourself!

**Docker Compose** 

Theoretically this will work in Docker however you will need to go in and change all the volumes to what you want them to be. 

**Networking**

You will need to port forward port 2001 and 17777 and set up NAT in your router. I used Pure NAT in my PFSense router for example set the NAT rules and then set the actual interface rules on WAN allowing the traffic to the WAN and the actual IP of the server. 

**Useful kubectl commands** 

To monitor the server install: 

kubectl -n reforger logs -l app=arma-reforger -c install-arma -f

To monitor the server itself:

kubectl -n reforger logs -l app=arma-reforger -c reforger -f

Checks A2S logs: 

kubectl -n reforger exec deploy/arma-reforger -c reforger -- sh -lc ' 
grep -R --line-number -E "Server registered|Direct Join Code|\[A2S\]" /home/profile/logs || true'
