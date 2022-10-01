# OSCP Bash Scripts

As part of the OSCP I found there were some manual steps I always carried out so I have written some quick and dirty bash scripts to speed up this manual process.

These scripts have a dependency on the outputs from Autorecon and **must be run from within the same folder the machines IP folder sits** that Autorecon creates. So if your scan is in /home/kali/Box/results/10.11.1.251 you run them from within that IP folder.

The two processes I found I carried out a lot were the following:

- Connect to every port manually with Netcat to see if there was something running that we are unaware of
- Connect to website pages to see if there is any data on them that we may be unaware of

To solve this I wrote the following scripts:

## 1. Connect with Netcat to all open ports & output result

````

for i in $(echo ${PWD##*/}|tr "," "\n"); do grep -E 'tcp\s+open' $(pwd)/scans/_full_tcp_nmap.txt | awk -F / '{print $1}' | awk 'BEGIN { ORS = "\n" } { print }' | xargs -i nc -w 3 -vn $i {}; done

````

## 2. Curl all open ports

```

for port in `grep -E 'tcp\s+open' $(pwd)/scans/_full_tcp_nmap.txt | awk -F / '{print $1}' | awk 'BEGIN { ORS = " " } { print }'`; do IP=${PWD##*/};RED='\033[0;31m';NC='\033[0m';echo ${RED}"http://"$IP":"$port;echo ${NC}; curl -m 1 $IP":$port"; printf "\n";printf "\n";printf "===============================";printf "\n"; done

```

## 3. Curl all website pages found on port 80 which is handy if not too many. Change the port to suit.

```

for link in `cat $(pwd)/scans/tcp80/tcp_80_http_feroxbuster_dirbuster.txt | awk -F ' ' '{print $5}' | awk 'BEGIN { ORS = " " } { print }'`; do RED='\033[0;31m';NC='\033[0m';echo ${RED}$link;echo ${NC}; curl -m 2 $link; printf "\n";printf "\n";printf "===============================";printf "\n"; done

```

## 4. Curl all website pages found but exclude pages that returned a 403

```

for link in `grep -v 403 $(pwd)/scans/tcp80/tcp_80_http_feroxbuster_dirbuster.txt | awk -F ' ' '{print $5}' | awk 'BEGIN { ORS = " " } { print }'`; do RED='\033[0;31m';NC='\033[0m';echo ${RED}$link;echo ${NC}; curl -m 2 $link; printf "\n";printf "\n";printf "===============================";printf "\n"; done

```

## 5. Curl the disallowed entries from robots.txt

```

for url in `grep -E 'Disallow:' robots.txt | awk '{print $2}'`; do RED='\033[0;31m';NC='\033[0m';echo ${RED}"http://"${PWD##*/}$url;echo ${NC};curl -s -m 2 ${PWD##*/}$url | grep -v "403 - Forbidden: Access is denied";printf "===============================";printf "\n"; done

```

## 6. Simply extract an open port list

```

grep -E 'tcp\s+open' $(pwd)/scans/_full_tcp_nmap.txt | awk -F / '{print $1}' | awk 'BEGIN { ORS = " " } { print }'

```
