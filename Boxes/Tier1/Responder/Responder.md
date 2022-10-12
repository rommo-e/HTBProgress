The responder box 📦  is a first challenge about LFI (Local File Inclusion), this vulnerability raises due to a lack of [[sanization]]  of the input when  and you can take advantage of the URL parameter known as [[page]] 

### Hosts
One approach is to see first what is on the IP address wether is a Server ,just a Site or a Web App. 

But we encountered with the problem that the IP address can't be resolved by our DNS, this is mainly caused to the management of 

So in order to be able to access to the IP we added the IP to the /etc/hosts file , remember to modify this file with root permisions .
Just add the IP and the domain that you're being redirected to, for this situation we have the unika.htb domain and the IP

`<bash/nano file:etc/hosts
#Host database
		{IP}       {Domain}
`


### Enumeration 
As usual we first enumerate using [[nmap]] 
`<bash>
$ sudo nmap -sC -sV -p- --min rate 5000 {IP}
`
While waiting for nmap to do the scanning it is always good to use the time for another enumeration process with [[gobuster]]

### LFI 
LFI occurs when an attacker is able to get a website to include a file that was not intended to be an option. 
A very common case is when an application uses the path to a file as input. 
Now we can check if this page is vulnerable to a Local File Inclusion or a Remote File Inclussion 


### Responder
One default tool in kali/parrot OS used for retrieving the hash number is redemer , this is a python script that listens on the port. It is needed to turn off the HTTP listening port option so it can listens through this port. 

This process is going to occur then when we turn off the http port option in the Redemer.config file 

`<bash> / nano Responder.conf
[Responder Core]
;Server to start
SQL = On
SMB = On
==HTTP = Off==
`

After doing this we can proceed starting the Responder with python3, using the flag -I to specify the network interface 

`<bash>
$sudo python3 Responder.py -I tun0
`
to find out what is the IP showed on tun0 remember to run $ifconfig

If this is not an option for obtain the hash then try to use the default tool installed on kali or parrot , this using the next command 

`<bash>
$sudo responder -I tun0
` 

Then crack the hash using john



#Installation 

`<bash>
$ git clone https://github.xom/lgandx/Responder
`



### John The Ripper
For cracking the founded hash we call for the tool john -> short for John the Ripper

-w : wordlist to use for cracking the hash 
`
$john -w=/usr/share/wordlists/rockyou.txt hash.txt 
`
#### Important Note
In case the file rockyou.txt which is a wordlist or dictionary is compressed in the file system we can decompress  the file using  [[gzip]]

`<bash> maybe for Debian based distros, if not search an alternative
$gzip -dk rockyou.txt.gz
`
-d is an option for restore the compressed file to its original state
-k is an option for keeping the file 


### WinRM
Now for trying to get a session on WinRM which is the service on the target running on port 

Since powershell is not installed on Linux we can use an alternate tool, this tool is Evil Winrm, is a tool created by Hackplayers , and it has it's own repository in Github. 

If WinRM it's not installed then use the command 

`<bash>
$sudo gem install evil-winrm
`

The command for establishing a connection to the remote endpoint is 

`<bash>
$evil-winrm -i {IP} -u administrator -p badminton
`
where -u is for the user 
and -p for the password cracked using John The Ripper

Then just search for the flag in the directory