# Anomalous DNS Activity Investigation

This page outlines the steps I took to investigate an alert triggered by **Anomalous DNS Activity** using various log files and tools. The investigation involved analyzing DNS queries, connection logs, and identifying IP addresses related to suspicious activity. Below, I document the process and results of each step in the investigation.

---

## Objective
To confirm if the alert related to **Anomalous DNS Activity** is a true positive by inspecting the PCAP file, DNS log file, and connection logs.

## Tools Used
- **Zeek** (formerly Bro)
- **PCAP file analysis**
- **Command-line utilities** such as `grep`, `sort`, `uniq`, and `wc`
- **Linux environment**

## Steps

### 1. Analyzing DNS records linked to the IPv6 address
The first task was to investigate the `dns-tunneling.pcap` file to determine how many DNS records were linked to an IPv6 address.

#### Command Used:
`zeek -Cr dns-tunneling.pcap`
`strings dns.log | zeek-cut qtype | grep '28' | wc -l`

The `AAAA` record type for IPv6 addresses corresponds to the number `28` in the query type (`qtype`) field. By counting the occurrences of this value, I determined the number of DNS records related to the IPv6 address.

#### Result:
- **Number of DNS records**: 320

![DNS record count](https://github.com/user-attachments/assets/91849397-725e-4b80-b13e-88beb67a1347)


---

### 2. Finding the longest connection duration
Next, I analyzed the `conn.log` file to identify the longest connection duration.

#### Command Used:
`cat conn.log | zeek-cut duration | sort -n`

Sorting the durations helped to find the maximum value.

#### Result:
- **Longest connection duration**: 9.420791 seconds

![Longest connection duration](https://github.com/user-attachments/assets/02b602e3-c2ca-4949-9f31-3ac79115c50c)


---

### 3. Counting unique DNS domain queries
I filtered the DNS log file for unique DNS domain queries.

#### Command Used:
`cat dns.log | zeek-cut query | rev | cut -d . -f 1-2 | rev | sort | uniq | wc -l`

This command allowed me to reverse domain names, extract the first two levels (e.g., second-level and top-level domains), and then count the unique queries.

#### Result:
- **Number of unique domain queries**: 6

![Unique domain queries](https://github.com/user-attachments/assets/8342c1e9-8caa-4e6a-8f29-ad8e008c5ac0)


---

### 4. Identifying the IP address of the source host
Finally, I examined the `conn.log` file to find the IP address of the source host associated with the abnormal DNS queries.

#### Command Used:
I used the connection logs to extract the relevant information from the logs.

#### Result:
- **IP address of the source host**: 10.20.57.3


---

## Conclusion
The investigation confirmed that the alert regarding **Anomalous DNS Activity** was a true positive. By analyzing the PCAP and log files using tools like Zeek, I was able to identify suspicious IPv6 DNS records, the longest connection duration, unique domain queries, and the source host IP. This provided valuable insights into the unusual network activity.

