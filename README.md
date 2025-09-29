## Project Overview
In this project, we will upload sample HTTP log files to Splunk SIEM and perform various analyses to gain insights into web server activity within the network.

## Prerequisites
Before starting the project, ensure the following:
- Splunk instance is installed and configured.
- HTTP log data sources are configured to forward logs to Splunk.

## Steps to Upload Sample HTTP Log Files to Splunk SIEM

### 1. Prepare Sample HTTP Log Files
- Obtain sample [HTTP log files](https://www.secrepo.com/maccdc2012/http.log.gz) in a suitable format (e.g., text files).
- Ensure the log files contain relevant HTTP events, including timestamps, request methods, URLs, response codes, user agents, etc.
- Save the sample log files in a directory accessible by the Splunk instance.

### 2. Upload Log Files to Splunk
- Log in to the Splunk web interface.
- Navigate to **Settings** > **Add Data**.
- Select **Upload** as the data input method.

### 3. Choose File
- Click on **Select File** and choose the sample HTTP log file you prepared earlier.

### 4. Set Source Type
- In the **Set Source Type** section, specify the source type for the uploaded log file.
- Choose the appropriate source type for HTTP logs (e.g., `access_combined` or a custom source type if applicable).

### 5. Review Settings
- Review other settings such as index, host, and sourcetype.
- Ensure the settings are configured correctly to match the sample HTTP log file.

### 6. Click Upload
- Once all settings are configured, click on the **Review** button.
- Review the settings one final time to ensure accuracy.
- Click **Submit** to upload the sample HTTP log file to Splunk.

### 7. Verify Upload
- After uploading, navigate to the search bar in the Splunk interface.
- Run a search query to verify that the uploaded HTTP events are visible.


## Steps to Analyze HTTP Log Files in Splunk SIEM


### 1. Search for HTTP Events
- Open Splunk interface and navigate to the search bar.
- Enter the following search query to retrieve HTTP events:
```
index=<your_http_index> sourcetype=<your_http_sourcetype>
```

### 2. Extract Relevant Fields
- Click on Extract New Fields in the Fields Tab.
- Choose a log entry as a sample and click next.
- Select the method to create fields i.e regular expression or delimiters (in this project i have used regular expression).
- Highlight or select the information and give a name to it, like for source IP you can highlight the source ip and give it a name as src_ip and then click create extraction.
- Validate whether the extracted field has been applied to every entries and click next.
- Save it.

### 3. Analyze Web Traffic Patterns
- Traffic Analysis to understand web traffic patterns.
```
index=<your_http_index> sourcetype=<your_http_sourcetype>
| stats count by src_ip, dst_ip | sort - count
```

- Determine the top or most frequent values using top limit
```
index=<your_http_index> sourcetype=<your_http_sourcetype>
| toplimit=10 src_ip, dst_ip | sort - count
```

- Identify top URLs or endpoints accessed by users.
```
index=<your_http_index> sourcetype=<your_http_sourcetype>
| top limit=10 uri
```

- Analyze response codes to identify errors or successful requests.
```
index=<your_http_index> sourcetype=<your_http_sourcetype>
| stats count by host, status_code | sort - count
```

### 4. Detect Anomalies
- Look for unusual patterns in file transfer activity.
```
index=<your_http_index> sourcetype=<your_http_sourcetype>
| timechart span=8h count by src_ip
```
- Analyze high volumes of error responses:
```
index=<your_http_index> sourcetype=<your_http_sourcetype>
| stats count by status
| where status >= 400
```

- Investigate file transfers to or from suspicious IP addresses.
```
index=<your_http_index> sourcetype=<your_http_sourcetype>
| search src_ip="suspicious_ip"    #Here you can write any suspicious ip that you think needs to be look after or searched
```


### 5. Analysis using visualization based commands
- Identify error rate trend
```
index=<your_http_index> sourcetype=<your_http_sourcetype>
| timechart span=5m count(eval(status_code>=400)) as errors, count as total
| eval error_rate = (errors*100)/total
```

- Identify Traffic Volume by method (Bar Chart)
```
index=<your_http_index> sourcetype=<your_http_sourcetype>
| chart count over method by status_code
```

- Identify Unique URis Over time
```
index=<your_http_index> sourcetype=<your_http_sourcetype>
| timechart span=4h dc(uri) as unique_uris    #dc basically finds distinct values i.e. unique
```

- Suspicious Scan Detection Dashboard Panel
```
index=<your_http_index> sourcetype=<your_http_sourcetype>
| stats count dc(uri) as unique_uris by src_ip
| where unique_uris > 100 OR count > 500
```

## Conclusion
Analyzing FTP log files using Splunk SIEM provides valuable insights into file transfer activities within a network. By monitoring FTP events, detecting anomalies, and correlating with other logs, organizations can enhance their security posture and protect against various cyber threats.


