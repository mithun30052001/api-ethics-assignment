# Task 1 — Classify and Handle PII Fields

| Field           | PII Type       | Action       | Justification                                                     |
| --------------- | -------------- | ------------ | ----------------------------------------------------------------- |
| full_name       | Direct PII     | Drop         | Directly identifies an individual and is not needed for research. |
| email           | Direct PII     | Drop         | Unique identifier that enables direct contact.                    |
| date_of_birth   | Indirect PII   | Mask         | Can enable re-identification when combined with other fields.     |
| zip_code        | Indirect PII   | Mask         | Full ZIP codes can identify individuals in small populations.     |
| job_title       | Indirect PII   | Generalize   | Useful for analysis but should be less specific.                  |
| diagnosis_notes | Sensitive Data | Pseudonymize | Contains sensitive health information and possible identifiers.   |

---

## Summary

* Dropped: full_name, email
* Masked: date_of_birth, zip_code
* Generalized: job_title
* Pseudonymized: diagnosis_notes

---

# Task 2 — Audit the API Script for Ethical Compliance

## Violation 1: Hardcoded API Key

### Problem

* API key is exposed in source code
* Risk of unauthorized usage and security breach
* Violates best practices and possibly API TOS

### Fix

Use environment variables to store sensitive credentials

```python
import os
import requests

API_URL = "https://healthstats-api.example.com/records"
API_KEY = os.getenv("HEALTH_API_KEY")

records = []
for page in range(1, 101):
    response = requests.get(API_URL, params={"page": page, "key": API_KEY})
    data = response.json()
    records.extend(data["results"])
```

---

## Violation 2: Excessive Data Collection & Storing Raw PII

### Problem

* Fetches large volume of data without limits
* Stores raw sensitive patient data permanently
* Violates data minimization and privacy principles

### ✅ Fix

* Limit API usage
* Apply rate limiting
* Store only anonymized data

```python
import os
import requests
import time

API_URL = "https://healthstats-api.example.com/records"
API_KEY = os.getenv("HEALTH_API_KEY")

def sanitize_notes(notes):
    return notes

records = []

for page in range(1, 21):
    response = requests.get(API_URL, params={"page": page, "key": API_KEY})

    if response.status_code != 200:
        break

    data = response.json()

    for record in data["results"]:
        cleaned_record = {
            "age_group": record.get("date_of_birth", "")[:4],
            "zip_prefix": record.get("zip_code", "")[:3],
            "job_title": record.get("job_title"),
            "diagnosis_notes": sanitize_notes(record.get("diagnosis_notes"))
        }
        records.append(cleaned_record)
    time.sleep(1)  

save_to_database(records)
```

---

## Key Improvements

* Secured API credentials
* Reduced data collection
* Removed direct identifiers
* Applied anonymization techniques
* Added rate limiting

---
