# 🤖 Job Application Automation — n8n Workflow

An intelligent, fully automated job application pipeline built with **n8n**. This workflow scrapes LinkedIn job listings, uses AI to filter roles that match your profile, tailors your resume for each position, and logs everything to a Google Sheet — all hands-free.

---

## ✨ What It Does

1. **Scrapes LinkedIn** for recent job postings matching your search query (up to 50 jobs at a time)
2. **Reads your resume** from a Google Doc
3. **AI-filters each job** — uses LLaMA 3.3 70B via Groq to decide if you're a good fit based on your resume vs. the job description
4. **Tailors your resume** — for every matched job, the AI rewrites your resume in HTML, optimized for ATS and aligned with that specific role
5. **Creates a new Google Doc** for each tailored resume, named after the company
6. **Logs to Google Sheets** — appends a row with job title, company, seniority, salary, applicant count, apply URL, and a direct link to the tailored resume

---

## 🔁 Workflow Architecture

```
Manual Trigger
    │
    ▼
Set LinkedIn Search URL
    │
    ▼
Scrape Jobs (Apify - LinkedIn Jobs Scraper)
    │
    ▼
Loop Over Items (batch size: 2)
    │
    ├──▶ Get Resume (Google Docs)
    │         │
    │         ▼
    │    AI: Check Job Fit (Groq - LLaMA 3.3 70B)
    │         │
    │         ▼
    │    Filter: verdict == true?
    │         │
    │         ▼
    │    AI: Tailor Resume → HTML (Groq - LLaMA 3.3 70B)
    │         │
    │         ▼
    │    Create Google Doc (CV - {Company Name})
    │         │
    │         ▼
    │    Upload Tailored Resume to Google Drive
    │         │
    │         ▼
    │    Append Row to Google Sheet (JOB LIST)
    │         │
    └──────────┘ (loop back)
```

---

## 🧰 Tech Stack & Services

| Service | Purpose |
|---|---|
| [n8n](https://n8n.io) | Workflow automation engine |
| [Apify](https://apify.com) — `linkedin-jobs-scraper` | LinkedIn job scraping |
| [Groq](https://groq.com) — LLaMA 3.3 70B | AI job filtering & resume tailoring |
| [Google Docs](https://docs.google.com) | Resume source & tailored output storage |
| [Google Sheets](https://sheets.google.com) | Job tracking dashboard |
| [Google Drive API](https://developers.google.com/drive) | Upload HTML resume content |

---

## ⚙️ Setup & Configuration

### 1. Prerequisites

- A running **n8n** instance (self-hosted or cloud)
- Accounts for: **Apify**, **Groq**, **Google** (Docs, Sheets, Drive)

### 2. Import the Workflow

1. Download `Job_Application_Automation.json`
2. In n8n, go to **Workflows → Import from File**
3. Select the JSON file

### 3. Connect Your Credentials

Set up the following credentials in n8n:

| Credential | Used By |
|---|---|
| Apify OAuth2 API | Scrape Jobs node |
| Groq API | Both AI nodes (job filter + resume tailor) |
| Google Docs OAuth2 | Get Resume, Create Document nodes |
| Google Sheets OAuth2 | Append Row node |
| Google Drive OAuth2 | HTTP Request (resume upload) node |

### 4. Configure the Workflow

**Set your LinkedIn search URL** in the `Edit Fields` node:
```
https://www.linkedin.com/jobs/search/?keywords=developer&f_TPR=r86400
```
Customize `keywords`, location filters, date range (`f_TPR`), and other LinkedIn URL parameters to match your search.

**Set your resume Google Doc URL** in the `Get a document` node — replace the existing document URL with the link to your own resume Google Doc.

**Set your Google Sheet ID** in the `Append row in sheet` node — replace the spreadsheet ID with your own job tracking sheet. The sheet should have these columns:

```
JOB TITLE | SENIORITY LEVEL | POSTED AT | COMPANY NAME | SALARY | DESCRIPTION | N*APPLICANT | APPLICATION URL | RESUME URL
```

---

## 🚀 Usage

1. Open the workflow in n8n
2. Click **"Execute Workflow"** to run manually
3. The workflow will process up to 50 jobs in batches of 2
4. Matching jobs will appear in your Google Sheet with links to their tailored resumes

---

## 📊 Output Example (Google Sheet)

| JOB TITLE | COMPANY NAME | SENIORITY LEVEL | SALARY | N°APPLICANTS | APPLICATION URL | RESUME URL |
|---|---|---|---|---|---|---|
| Software Engineer | Acme Corp | Mid-Senior | $120k | 47 | [Apply](https://...) | [CV](https://docs.google.com/...) |

---

## ⚠️ Important Notes

- **API keys in the JSON**: Before sharing or committing this file, make sure to **remove any hardcoded API keys** from the HTTP Request nodes. Credentials should always be managed through n8n's credential manager.
- **AI-generated resume content**: The resume tailoring prompt allows the AI to enhance or infer experience to better match jobs. Review all generated resumes before submitting applications.
- **LinkedIn scraping**: Use responsibly and in accordance with LinkedIn's Terms of Service. The Apify actor handles rate limiting, but avoid running too frequently.
- **Groq rate limits**: The free tier of Groq has rate limits. If you're processing many jobs, consider adding a `Wait` node between batches.

---

## 🛠️ Customization Ideas

- **Change the AI model**: Swap `llama-3.3-70b-versatile` for another Groq-supported model (e.g., `mixtral-8x7b-32768`)
- **Add email notifications**: Insert a Gmail or SMTP node to get notified when a new matching job is found
- **Schedule it**: Replace the Manual Trigger with a Schedule Trigger to run daily automatically
- **Add more job sources**: Extend with Indeed, Glassdoor, or other job board scrapers

---

## 📄 License

MIT — feel free to fork, modify, and share.
