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


