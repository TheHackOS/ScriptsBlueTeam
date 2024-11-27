## SPLUNK

### Sintaxis

+ Fields command is used to add or remove mentioned fields from the search results. To remove the field, minus sign ( - ) is used before the fieldname and plus ( + ) is used before the fields which we want to display.

~~~bash
index=windowslogs | fields + host + User - SourceIp
~~~

+ This command is used to search for the raw text while using the chaining command |

~~~bash
index=windowslogs | search Powershell
~~~

+ Dedup is the command used to remove duplicate fields from the search results. We often get the results with various fields getting the same results. These commands remove the duplicates to show the unique values.

~~~bash
index=windowslogs | table EventID User Image Hostname | dedup EventID
~~~

+ It allows us to change the name of the field in the search results. It is useful in a scenario when the field name is generic or log, or it needs to be updated in the output.

~~~bash
index=windowslogs | fields + host + User + SourceIp | rename User as Employees
~~~

+ Each event has multiple fields, and not every field is important to display. The Table command allows us to create a table with selective fields as columns.

~~~bash
index=windowslogs | table EventID Hostname SourceName
~~~

+ The head command returns the first 10 events if no number is specified.
+ head 10 - will return the top 10 events from the result list

~~~bash
index=windowslogs |  table _time EventID Hostname SourceName | head 5
~~~

+ The Tail command returns the last 10 events if no number is specified.

~~~bash
index=windowslogs |  table _time EventID Hostname SourceName | tail 5 
~~~

The Sort command allows us to order the fields in ascending or descending order.

~~~bash
index=windowslogs |  table _time EventID Hostname SourceName | sort Hostname
~~~

+ The reverse command simply reverses the order of the events.

~~~bash
index=windowslogs | table _time EventID Hostname SourceName | reverse
~~~

+ This command returns frequent values for the top 10 events

~~~bash
index=windowslogs | top limit=7 Image
~~~

+ This command does the opposite of top command as it returns the least frequent values or bottom 10 results.

~~~bash
index=windowslogs | rare limit=7 Image
~~~

+ The highlight command shows the results in raw events mode with fields highlighted. 

~~~bash
index=windowslogs | highlight User, host, EventID, Image
~~~

+ The chart command is used to transform the data into tables or visualizations.

~~~bash
index=windowslogs | chart count by User
~~~

+ The timechart command returns the time series chart covering the field following the function mentioned. Often combined with STATS commands.

~~~bash
index=windowslogs | timechart count by Image
~~~

### Analizando malware Dridex con splunk

+ Extraer campos del archivo pcap para analizar

~~~java
tshark -r pcap/ursnif.pcap -T fields -e frame.time_utc -e ip.src -e tcp.srcport -e udp.srcport -e ip.dst -e tcp.dstport -e udp.dstport -e tcp.len -e dns.id -e dns.flags -e dns.qry.name -e frame.protocols -e http.request.method -e http.user_agent -e http.request.full_uri -e http.host -E header=y -E separator=/t > output.csv
~~~

[+] Analisis

+ Conteo de resolución DNS maliciosa en cada dominio
~~~java
source="output.csv" host="unsrif" index="bad_traffic" sourcetype="unsrif_mlw" dns_qry_name=* | search dns_qry_name IN ("jandneneet.com", "ceugaylordwinifred.com", "nlourdesprice.com", "sbrian025ao.com", "btyr34brian.com", "mccannmia.com") | table dns_qry_name | chart count by dns_qry_name
~~~

+ Busqueda de host maliciosos
~~~java
source="output.csv" host="unsrif" index="bad_traffic" sourcetype="unsrif_mlw" http_request_method=GET | table ip_src ip_dst http_host http_request_full_uri http_request_method | search http_host IN ("jandneneet.com", "ceugaylordwinifred.com", "nlourdesprice.com", "sbrian025ao.com", "btyr34brian.com", "mccannmia.com")
~~~

+ Trafico malicioso de ip maliciosas
~~~java
source="output.csv" host="unsrif" index="bad_traffic" sourcetype="unsrif_mlw" | search ip_src IN ("91.211.249.204","162.213.37.188","216.99.151.165") | table ip_src | chart count by ip_src
~~~