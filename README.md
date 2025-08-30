# **Splunk Analyst**


## **Overview**
This project highlights my experience using **Splunk Enterprise** with various datasets to simulate SOC analyst workflows. After installing Splunk locally and ingesting the data, I applied discovery techniques like `metadata`, `metasearch`, `fieldsummary`, and regex searches to uncover insights. To streamline investigations, I built a reusable macro for regex exploration, then created targeted searches such as identifying **CloudTrail API activity without MFA**. I also analyzed **HTTP logs** to extract methods, status codes, top talkers, error rates, and host-to-host flows. Finally, I built two dashboards—**Host Authentication Attempts** and **HTTP Logs Overview**—to clearly present authentication trends and web traffic behaviors, demonstrating my ability to investigate, analyze, and visualize security data in Splunk.





## **Project Steps**
1.  **Import Dataset**  
   - Add data ➜ Upload ➜ Select your BOSSv3 files (e.g., JSON, CSV, or raw logs) ➜ Set `index = botsv3`.  
   - Set the time range to **All time** and verify ingestion with:
     ```spl
     index=botsv3 earliest=0

2. **Listed all sourcetypes within the dataset**
   ```spl
     | metadata type=sourcetypes index=botsv3
     | stats values(sourcetype)

<img width="1911" height="933" alt="image" src="https://github.com/user-attachments/assets/9a9ed207-1f74-4672-9c2a-6afe4640393a" />






3. **Using meta Search**
   - Metasearch allows you to run a less intensive search because it only scans the metadata index rather than every event.
   - The metasearch allowed me to search all AWS-related events, using the wildcard at  the end of the query 
   ```spl
     | metasearch index=botsv3 sourcetype=aws*
<img width="956" height="467" alt="image" src="https://github.com/user-attachments/assets/45003bf4-e9f2-43a3-a51c-64b73a871433" />




4. **Created a field Summary**
   -    Created a field summary for the aws:cloudtrail sourcetypes, which allows you to produce stats about all fields and events.
   -    The head command limits the results to the first 1000 events
   -    | search values!="[]" = removes all empty fields
   -    | fields field values = Keeps just the field name (field) and its observed values.
   -    | rex field=values max_match=0 "\{\"value\":\"(?<extracted_values>[^\"]+)\"" =
        Looks inside the values field (which stores data as JSON-like structures).
        Extracts the actual value string inside "value":"...".
        max_match=0 means capture all matches, not just the first one.
        The results get stored in a multivalue field called extracted_values.
   -    | fields field extracted_values = Clean up again: Keeps only the field name and the extracted values.
   -    | eval extracted_values = mvdedup(extracted_values) =
        Removes duplicate values from the extracted_values multivalue field, leaving only unique entries.



   ```spl
     |index=botsv3 sourcetype=aws:cloudtrail
     | head 1000
     | fieldsummary
     | search values!="[]"
     | fields field values
     | rex field=values max_match=0 "\{\"value\":\"(?<extracted_values>[^\"]+)\""
     | fields field extracted_values
     | eval extracted_values = mvdedup(extracted_values)

<img width="959" height="472" alt="image" src="https://github.com/user-attachments/assets/2f445265-c48d-4bff-b180-16104bd0bfb0" />




5. **Created macro**
   - Allowed me to run the command without having to type the query out every time.
   - Advance search / search macros / click the new search macro button/ name and add query
  
<img width="715" height="334" alt="image" src="https://github.com/user-attachments/assets/bf08014e-079a-4be8-ac86-c18d98fca77a" />
   - The search field of anything with any values that have mfa within the field allowed me to narrow it down to 


<img width="956" height="224" alt="image" src="https://github.com/user-attachments/assets/c9b9e4fd-d29b-4977-8dc4-4060a7cf0c6e" />


 6. **Deeper Dive for the flag**
  - With the two fields narrowed down I was able to confirm where I can find API activity that took place without MFA 
(Field = userIdentity.sessionContext.attributes.mfaAuthenticated)


<img width="956" height="215" alt="image" src="https://github.com/user-attachments/assets/4d55b6c4-ea23-4d95-9411-7613f6314a91" />

 - Was able to find a field under the name hardware, then did a metasearch to see what could be found under hardware.
<img width="952" height="398" alt="image" src="https://github.com/user-attachments/assets/ddbd9b6c-3cdc-4362-be48-0ed502282aea" />


<img width="955" height="473" alt="image" src="https://github.com/user-attachments/assets/ba27f29b-b1c3-4114-b89f-ecce61c35a25" />


---------------------------------------------------------------------------------------------------------------


## **HTTP Log Analysis**

1. **Created New Fields**
- extracting fields from within the data to make parsing through the data easier and more precise
- Specified the source and destination ip addresses. 
- Method =  Http Request Methods
- Status = Outcome of the http request
  
<img width="950" height="464" alt="image" src="https://github.com/user-attachments/assets/0a1da51e-a824-4af7-8444-eba8590b4472" />

<img width="292" height="244" alt="image" src="https://github.com/user-attachments/assets/2cdd94cd-6a77-4a01-8d56-84d6d39bb1f6" />


<img width="299" height="275" alt="image" src="https://github.com/user-attachments/assets/a5050104-2e3c-4a0c-aa8d-82cbca60c94e" />


2. **Created Table**
- I built a Splunk search that extracts and displays only the Source IP,  Destination IP fields, Ports.
 ```spl
     index=_* OR index=* sourcetype=HTTPlogs | table src_ip, src_port, dest_ip, dest_port
```

<img width="954" height="404" alt="image" src="https://github.com/user-attachments/assets/c2360733-3b2e-4ba8-aa4d-71762c0102da" />


3. **investiagting IPs**
- These are the top 20 destination ip addresses that were requested and how many times they were requested
  ```spl
      index=_* OR index=* sourcetype=HTTPlogs | top limit=20 dest_ip
  ```

<img width="954" height="404" alt="image" src="https://github.com/user-attachments/assets/fb6017f7-f0fc-40df-aab9-d15c993e24c7" />



- The following search shows which source ips interacted with destination 192.168.21.102 the most.

  ```sql
  index=_* OR index=* sourcetype=HTTPlogs dest_ip= "192.168.21.102" | top limit=10 src_ip
  ```

<img width="959" height="278" alt="image" src="https://github.com/user-attachments/assets/eb8b8eff-3750-4a88-8651-cfffc1078ba5" />


- This allowed me to see the direct HTTP Status/Response codes

```sql
index=_* OR index=* sourcetype=HTTPlogs | stats count by status | where status >=400
```
  

