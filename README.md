# Job Scraping and Refinement with OpenAI GPT

This Python script scrapes job postings from multiple job sites, structures the data, and refines it using OpenAI's GPT model. The final output is a JSON file containing structured and refined job listings.

## Table of Contents
1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Steps](#steps)
   - [Step 1: Define Job Sites to Scrape](#step-1-define-job-sites-to-scrape)
   - [Step 2: Scrape Jobs from Each Site](#step-2-scrape-jobs-from-each-site)
   - [Step 3: Convert to Structured Format](#step-3-convert-to-structured-format)
   - [Step 4: Send Data to OpenAI GPT for Refinement](#step-4-send-data-to-openai-gpt-for-refinement)
4. [Output](#output)
5. [Conclusion](#conclusion)

## Overview
This script performs the following tasks:
1. Scrapes job postings from multiple job sites.
2. Structures the scraped data into a consistent format.
3. Sends the structured data to OpenAI GPT for refinement (e.g., filling in missing experience levels, ensuring consistency in job nature).
4. Saves the refined job listings in a JSON file.

## Prerequisites
Before running the script, ensure you have the following:
- Python >=3.10 installed.
- Required Python libraries: `jobspy`, `openai`, `csv`, `json`.
- An OpenAI API key (set in the `openai.api_key` variable).

You can install the required libraries using pip:
```bash
pip install jobspy openai
```

## Steps

### Step 1: Define Job Sites to Scrape
The script scrapes job postings from the following sites:
- LinkedIn
- Indeed
- Zip Recruiter
- Google Jobs
- Bayt

```python
sites = ["linkedin", "indeed", "zip_recruiter", "google", "bayt"]
```

### Step 2: Scrape Jobs from Each Site
The script scrapes 20 job postings from each site for the search term "software engineer" in "Islamabad, Pakistan". The jobs must be no older than 72 hours.

```python
all_jobs = []
for site in sites:
    jobs = scrape_jobs(
        site_name=[site],
        search_term="software engineer",
        location="Islamabad, Pakistan",
        results_wanted=20,
        hours_old=72,
        country_indeed="Pakistan",
    )
    all_jobs.extend(jobs.to_dict(orient="records"))
```

### Step 3: Convert to Structured Format
The scraped job data is converted into a structured format with the following fields:
- `job_title`
- `company`
- `experience` (initially set to "N/A")
- `jobNature` (onsite or remote)
- `location`
- `salary`
- `apply_link`

```python
formatted_jobs = []
for job in all_jobs:
    formatted_jobs.append({
        "job_title": job.get("title", "N/A"),
        "company": job.get("company", "N/A"),
        "experience": "N/A",  # Experience will be filled by GPT
        "jobNature": "onsite" if "onsite" in job.get("location", "").lower() else "remote",
        "location": job.get("location", "N/A"),
        "salary": job.get("salary", "N/A"),
        "apply_link": job.get("job_url", "N/A"),
    })
```

The raw job data is saved to a CSV file for debugging purposes.

```python
with open("raw_jobs.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=formatted_jobs[0].keys())
    writer.writeheader()
    writer.writerows(formatted_jobs)
```

### Step 4: Send Data to OpenAI GPT for Refinement
The structured job data is sent to OpenAI GPT for refinement. The GPT model is instructed to:
1. Fill in missing experience levels based on job titles.
2. Ensure consistency in job nature (onsite, remote, hybrid).
3. Format the data in a specific JSON structure.

```python
openai.api_key = "your_openai_api_key_here"

prompt = f"""
The following is a list of job postings. Please:
1. Fill in missing experience levels based on job titles.
2. Ensure consistency in job nature (onsite, remote, hybrid).
3. Format in the following JSON format:

{{
 "relevant_jobs": [
 {{
 "job_title": "Full Stack Engineer",
 "company": "XYZ Pvt Ltd",
 "experience": "2+ years",
 "jobNature": "onsite",
 "location": "Islamabad, Pakistan",
 "salary": "100,000 PKR",
 "apply_link": "https://linkedin.com/job123"
 }}
 ]
}}

Here is the raw job data:
{formatted_json}
"""

response = openai.ChatCompletion.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": prompt}],
    temperature=0.2,
)

structured_jobs = response["choices"][0]["message"]["content"]
```

### Output
The refined job listings are saved in a JSON file named `structured_jobs.json`.

```python
with open("structured_jobs.json", "w", encoding="utf-8") as f:
    f.write(structured_jobs)

print("✅ Formatted job listings saved to 'structured_jobs.json'.")
```

## Conclusion
This script automates the process of scraping job postings, structuring the data, and refining it using OpenAI GPT. The final output is a clean and structured JSON file containing job listings, ready for further analysis or integration into other systems.
