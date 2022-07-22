# OSCP Bash Scripts

As part of the OSCP I found there were some manual steps I always carried out so I have written some quick and dirty bash scripts to speed up this manual process.

These scripts have a dependency on the outputs from Autorecon and **must be run from within the same folder your 'results' folder sits** that Autorecon creates.

The two processes I found I carried out a lot were the following:

- Connect to every port manually with Netcat to see if there was something running that we are unaware of
- Connect to website pages to see if there is any data on them that we may be unaware of

To solve this I wrote the following scripts:

## Netcat open port query

````
for i in $(echo ${PWD##*/}|tr "," "\n"); do grep -E 'tcp\s+open' $(pwd)/results/10.11.1.71/scans/_full_tcp_nmap.txt | awk -F / '{print $1}' | awk 'BEGIN { ORS = "\n" } { print }' | xargs -i nc -w 3 -vn $i {}; done
````

## Curl all open ports

```
for port in `grep -E 'tcp\s+open' $(pwd)/results/10.11.1.71/scans/_full_tcp_nmap.txt | awk -F / '{print $1}' | awk 'BEGIN { ORS = " " } { print }'`; do IP=${PWD##*/};RED='\033[0;31m';NC='\033[0m';echo ${RED}"http://"$IP":"$port;echo ${NC}; curl -m 1 $IP":$port"; printf "\n";printf "\n";printf "===============================";printf "\n"; done

```

## Curl all website pages found on port 80 (change port to suit)

```
for link in `cat $(pwd)/results/${PWD##*/}/scans/tcp80/tcp_80_http_feroxbuster_dirbuster.txt | awk -F ' ' '{print $5}' | awk 'BEGIN { ORS = " " } { print }'`; do IP=10.11.1.71;RED='\033[0;31m';NC='\033[0m';echo ${RED}$link;echo ${NC}; curl -m 1 $link; printf "\n";printf "\n";printf "===============================";printf "\n"; done
```

## Curl all website pages found but exclude pages that retured a 403

```
for link in `grep -v 403 $(pwd)/results/${PWD##*/}/scans/tcp80/tcp_80_http_feroxbuster_dirbuster.txt | awk -F ' ' '{print $5}' | awk 'BEGIN { ORS = " " } { print }'`; do IP=10.11.1.71;RED='\033[0;31m';NC='\033[0m';echo ${RED}$link;echo ${NC}; curl -m 1 $link; printf "\n";printf "\n";printf "===============================";printf "\n"; done
```

## Simply extract an open port list

```
grep -E 'tcp\s+open' $(pwd)/results/10.11.1.71/scans/_full_tcp_nmap.txt | awk -F / '{print $1}' | awk 'BEGIN { ORS = " " } { print }'
```
