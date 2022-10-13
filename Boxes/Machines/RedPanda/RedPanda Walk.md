### Introduction 
The box is vulnerable to injection attacks, so we can tink first off the SQL injection attack, for a first try analyze the technologies used on the page with wapalyzer, it is always very useful. 


The first thing to do in this box is to add the IP address with its domain into the file /etc/hosts 

### Connection
The IP address of this vulnerable machine is 10.10.11.170 
and the domain is redpanda.htb

`<bash>
#HTB
	10.10.11.170         redpanda.htb
`

Rather than connecting inmediatly we can comprobate the connection sending 1 ping , this is for practical purposes since ping will continue until we stop the process , for ping 1 time run the next command on the terminal after adding the IP address with its respective domain. 

`<bash>
$ping -c 1 redpanda.htb
#the -c option is for count
`

### Enumeration 

For a quick scanning of all the ports set the min rate to 5000 running the next command 

`<bash>
$ sudo nmap -sS --open -p- --min-rate 5000 -n -Pn -vvv redpanda.htb -oG allPorts
#-p- option is for all the ports 
#--min-rate is for the quantity of packets send every second
`

Try out to make an script to run this options automatically called nmapfast.sh

Once we obtain information about the open ports  we can scan the ports to obtain more information about the OS or extra details about the services running on these ports 

` <bash> 
$sudo nmap -p22,8080 redpanda.htb -oN targeted
`

Here we obtain important information about how the http page tittle was made , since we can read that 

8080/tcp open http-proxy 
http-tittle {sometitle}  | Made with Spring Boot 

We can use this information for searching exploits 

Another way to obtain this kind of information instead of running another nmap scan is to use the tool whatweb we specify the port 

`<bash>
whatweb http://redpanda.htb:8080/
`


### Browsing the site 

If we already finished to scan 
