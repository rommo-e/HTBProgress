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

If we already finished to scan the open ports now we can navigate to the site , we firs see a big picture of a red panda and below we can see a navigation bar. 
Nothing comes back when we use our tool Wappalyzer , no technologies are found just a common site. 

But the result is not the common one since we search common words and nohting appear until we just click on the search icon without typing any word. 
Then we start to search for code or see if we can execute javascript from the bar .


In this case now we can see a clue about injection attacks

### Find new paths
So since we only have access to the search bar the first thing that comes to mind is to look out for extra hidden pages in the web page , this can be done with wfuzz or gobuster. 
This is going to help us to gather more information that can be obtained from these hidden pages.

The first alternative is wfuzz and to look out for these paths we would need to use 
`<bash>
$wfuzz -c -t 200 --hc=404 -w /usr/share/wordlists/dirb/dirbuster/directory-list-lowercase-2.3-medium.txt http://redpanda.htb:8080/FUZZ
`

Similarly with gobuster 

`<bash>
$ gobuster dir --url http://{target_IP}/ --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php,html
`

Whichever is the tool we chosed to use we can get a name about a path that is called /stats

Within this directory we can now find names about the authors of the site  and we are allowed to export the data about their images and results on the site. 


### First memo 

So far so good we have been able to find hidden sites that provided us with information about the authors and stats, we have a clue about what could we do in the site making an injection attack but the most considerable clue that we have is the Spring Bot label about how the site was made. 

We can now search for exploits for this bot
