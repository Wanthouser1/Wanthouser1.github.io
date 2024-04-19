---
layout: page
title: Automated Referral Data Collection and Export
description: Data collection from eRelo's API
img: assets/img/7.jpg
importance: 3
category: work
---

This Python script facilitates the automated collection and export of referral data from an external API. The script utilizes the requests library to fetch data from the eRelocation API at regular intervals. Upon retrieval, the data is processed and stored in a Pandas DataFrame. Subsequently, the script exports the DataFrame to a CSV file named "referrals_data.csv" for further analysis or integration with other systems. Additionally, the script is configured to run on a predefined schedule using the schedule library, ensuring seamless and periodic updates of the referral dataset. By automating this process, the script enables efficient data management and analysis for eRelocation's referral program.

{% raw %}
```python
import json 
import requests
import pandas as pd
import schedule
import time

def eRelo_schedule():
    parameters = {
        "CreatedAfterUTC": "2020-01-01T00:00:00",
        "RecordsPerPage": 2000
    }
    
    headers = {
        "Content-Type": "application/json",
        "Authorization": str('AUTHORIZATION_TOKEN'),
    }

    referrals = []

    pagenum = 1

    while True:
        url = f"https://restapi.erelocation.net/api/v1/GetReferrals?PageNum={pagenum}"
        print("Requesting", url)
        resp = requests.get(url, headers=headers, params=parameters)
        data = resp.json()
        
        # Check if there are no more referrals
        if len(data.get('Referral', [])) == 0:
            print("All referrals fetched.")
            break
        
        referrals.extend(data['Referral'])
        pagenum += 1

    # Convert the list of dictionaries to a DataFrame
    df = pd.DataFrame(referrals)
    
    # Export DataFrame to a CSV file
    csv_file = "referrals_data.csv"
    if not os.path.isfile(csv_file):
        df.to_csv(csv_file, index=False)
        print(f"CSV file '{csv_file}' created successfully.")
    else:
        df.to_csv(csv_file, mode='a', header=False, index=False)
        print(f"Data appended to CSV file '{csv_file}'.")

# Schedule the function to run every 24 hours
schedule.every(24).hours.do(eRelo_schedule)

# Keep the script running to allow scheduling
while True:
    schedule.run_pending()
    time.sleep(1)
```

{% endraw %}
