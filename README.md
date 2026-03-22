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
