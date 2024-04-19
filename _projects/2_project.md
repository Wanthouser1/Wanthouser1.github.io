---
layout: page
title: Data Privacy Scanner
description: Personal Identifiable Information (PII) Identifier
img: assets/img/3.jpg
importance: 2
category: work
giscus_comments: false
---
This Python script demonstrates the implementation of a data privacy scanner using Apache Spark on Databricks. The script integrates with various tools and libraries to analyze text for personally identifiable information (PII) across different data catalogs and tables.

Key features include:

Catalog Selection: Enables users to select specific catalogs and entities to scan for PII, providing flexibility in targeting data sources.

Text Analysis: Utilizes the Presidio AnalyzerEngine to analyze text for PII, leveraging natural language processing techniques.

Concurrent Scanning: Employs concurrent execution with ThreadPoolExecutor to scan and tag tables in parallel, optimizing performance.

Interactive Display: Presents scan results in an interactive DataFrame format, facilitating easy interpretation and further analysis.

By deploying this script, organizations can enhance their data governance practices by proactively identifying and managing sensitive information, thereby mitigating risks associated with data privacy compliance.

{% raw %}
```python
# Install required Python packages from requirements.txt file
%pip install -q -r ../../requirements.txt

# Restart Python to ensure the installed packages are available
dbutils.library.restartPython()

# Download and install the English language model for spaCy
%sh
python -m spacy download en_core_web_lg > /databricks/driver/logs/spacy.log

# Import privacy functions from a common script
%run ../../common/privacy_functions

# Retrieve a list of all available catalogs
all_catalogs = list(filter(None, [x[0] for x in sql("SHOW CATALOGS").limit(1000).collect()]))

# Copy the list of all catalogs and insert "ALL" as the first option
catalogs = all_catalogs.copy()
catalogs.insert(0, "ALL")

# Retrieve a list of all supported entities
supported_entities = all_supported_entities.copy()
supported_entities.insert(0, "ALL")

# Create multi-select widgets for choosing catalogs, entities, and language
dbutils.widgets.multiselect(name="catalogs", defaultValue="ALL", choices=catalogs, label="catalogs_to_scan")
dbutils.widgets.multiselect(name="entities", defaultValue="ALL", choices=supported_entities, label="entities_to_detect")
dbutils.widgets.multiselect(name="language", defaultValue="en", choices=["en"], label="language")

# Retrieve selected options from widgets
catalogs = tuple(get_selection(selection=dbutils.widgets.get("catalogs").split(","), all_options=all_catalogs))
entities = get_selection(selection=dbutils.widgets.get("entities").split(","), all_options=all_supported_entities)
language = dbutils.widgets.get("language")

# Import AnalyzerEngine from presidio_analyzer module
from presidio_analyzer import AnalyzerEngine

# Broadcast the AnalyzerEngine instance to all nodes in the cluster
broadcasted_analyzer = sc.broadcast(AnalyzerEngine())

# Define a function to analyze text using the AnalyzerEngine
# In an ideal world we would define the UDFs in the class, but a Spark UDF can only be defined in a class as a static method...
def analyze_text(text: str) -> str:
    analyzer = broadcasted_analyzer.value
    analyzer_results = analyzer.analyze(text=text, entities=entities, language=language)
    return json.dumps([x.to_dict() for x in analyzer_results]) 

# Define a function to apply analyze_text function to a pandas Series
def analyze_series(s: pd.Series) -> pd.Series:
    return s.astype(str).apply(analyze_text)

# Define a Pandas UDF for analyzing text in a DataFrame
analyze_udf = pandas_udf(analyze_series, returnType=StringType())

# Create a PIIScanner instance with specified parameters
pii_scanner = PIIScanner(spark=spark, broadcasted_analyzer=broadcasted_analyzer, entities=entities, language=language,  sample_size=1000, average_score=0.5, hit_rate=60)

# Display all tables in a DataFrame
all_tables = pii_scanner.get_all_uc_tables(spark=spark, catalogs=catalogs).where("table_schema != 'information_schema'")
display(all_tables)

#Create DataFrame with fake PII data
df = generate_fake_pii_data(num_rows=1000).select("customer_id", "name", "email", "ssn", "iban", "credit_card", "phone_number", "date_of_birth", "ipv4", "ipv6", "freetext")
display(df)

#test scan 
test_scan = df.transform(pii_scanner.scan_dataframe)
display(test_scan)

# Import required modules and libraries
import os
import concurrent.futures

# Initialize an empty DataFrame to store scan results
scan_results = pd.DataFrame()

# Use ThreadPoolExecutor to concurrently scan and tag tables
with concurrent.futures.ThreadPoolExecutor(max_workers=os.cpu_count()) as executor:
  # Submit tasks for each table in parallel
  futures = [executor.submit(pii_scanner.scan_and_tag_securable, f"{securable.table_catalog}.{securable.table_schema}.{securable.table_name}", securable.table_type) for securable in all_tables.collect()]
  # Process completed tasks
  for future in concurrent.futures.as_completed(futures):

    result = future.result()
    if isinstance(result, pd.DataFrame):
      scan_results = pd.concat([scan_results, result])
    else:
      # Print any errors or messages
      print(result)

# Display scan results DataFrame
display(scan_results)

{% endraw %}
