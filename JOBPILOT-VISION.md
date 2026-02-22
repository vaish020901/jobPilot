# JobPilot — Complete Project Vision & Build Guide

## WHAT IS JOBPILOT?

JobPilot is an AI-powered job application automation tool built for the Indian job market. It automatically searches for jobs on Naukri.com, intelligently filters them against the user's profile, tailors their resume for each matching job using AI, and applies — all automatically.

**One sentence:** "You sleep, JobPilot applies to your dream jobs with a perfectly tailored resume."

---

## THE PROBLEM WE'RE SOLVING

Indian job seekers face a brutal reality:
- Applying to 50-100 jobs manually takes 5-10 hours per day
- Every job needs a slightly different resume emphasis
- Copy-pasting the same resume everywhere gets low response rates
- Existing tools (LazyApply, Sonara) don't support Naukri and are expensive ($99/year)
- No tool exists that tailors resumes per job AND auto-applies on Indian platforms

---

## THE COMPLETE USER FLOW

Step 1: CONFIGURATION (one-time setup)
- Enter Naukri email & password (in a normal form, NOT secrets/env vars)
- Upload master resume (PDF)
- Enter Gemini API key (free)
- Set job preferences:
  - Job titles: "Software Engineer, Python Developer"
  - Locations: "Bangalore, Hyderabad, Remote"
  - Experience: 0-3 years
  - Expected salary range
  - Key skills: "Python, React, Node.js"
- Set match threshold: 70% (only apply if score > 70)
- Set daily limit: 30 applications/day

Step 2: CLICK "ENGAGE"

Step 3: JOBPILOT DOES EVERYTHING AUTOMATICALLY:

A. LOGIN TO NAUKRI
   Playwright opens browser → logs into Naukri.com

B. SEARCH FOR JOBS
   For each job title + location combination:
   → Search "Software Engineer in Bangalore"
   → Search "Python Developer in Hyderabad"
   → Search "Software Engineer Remote" ... etc

C. EXTRACT JOB LISTINGS
   For each search result page:
   → Scrape all job cards (title, company, desc)
   → Click into each job to get FULL description
   → Extract: skills needed, experience, salary, responsibilities, qualifications

D. AI MATCHING (Gemini API)
   For each extracted job:
   → Send to Gemini: job description + candidate resume
   → Gemini returns: { score: 82, reasons: [...] }
   → If score < threshold (70) → SKIP this job
   → If score >= threshold → PROCEED to tailoring

E. AI RESUME TAILORING (Gemini API)
   For each matched job (score >= 70):
   → Send to Gemini: original resume + job description
   → Gemini tailors: rewrite summary, reorder skills, highlight relevant experience, add keywords
   → Keep it TRUTHFUL — don't invent anything
   → Generate a tailored PDF resume from AI output

F. AUTO-APPLY
   For each matched + tailored job:
   → Click "Apply" on Naukri
   → Wait 2-5 seconds (anti-detection)
   → Fill in form fields (name, email, phone)
   → Upload the TAILORED resume (not the original!)
   → Generate cover letter via AI if required
   → Submit application
   → Wait 30-90 seconds before next
   → Take 2-5 min break every 5-10 applications

G. TRACK & LOG
   → Save to database: job title, company, match score, tailored resume used, timestamp, status
   → Update dashboard in real-time

Step 4: USER WATCHES DASHBOARD
- Real-time: "Applying to Job 12/25..."
- Match scores for each job
- Which resume version was used
- Application status tracking
- Daily/weekly statistics

---

## TECH STACK

- Frontend/Dashboard: React + TypeScript + Tailwind (already built in Replit)
- Backend API: Node.js/Express or Python Flask
- Browser Automation: Playwright (controls real browser on Naukri)
- AI Engine: Google Gemini API (free tier)
- Resume Generation: python-docx or ReportLab (creates tailored PDFs)
- Database: SQLite or PostgreSQL
- File Storage: Local /uploads folder

---

## CONFIGURATION PAGE REQUIREMENTS

Naukri Credentials:
- Email (text input)
- Password (password input, masked, encrypted in DB)
- "Test Connection" button (verifies login works)

Resume:
- Upload Master Resume (PDF drag-and-drop)
- Preview uploaded resume
- Resume text extraction (parse PDF to text for AI)

Job Preferences:
- Job Titles (tag input, add multiple)
- Locations (tag input, add multiple)
- Min Experience (number)
- Max Experience (number)
- Min Salary (optional, number)
- Key Skills (tag input)

Automation Settings:
- Match Score Threshold (slider, 0-100, default 70)
- Max Applications Per Day (number, default 30)
- Dry Run Mode (toggle — does everything except submit)
- Auto-generate cover letters (toggle)

API Settings:
- Gemini API Key (text input)
- Link: "Get free API key here"

---

## DASHBOARD PAGE REQUIREMENTS

Command Center:
- ENGAGE button (starts automation)
- ABORT button (stops immediately)
- Status indicator (Idle / Running / Paused)

Real-time Progress (when running):
- Current action: "Scanning jobs for Software Engineer in Bangalore..."
- Progress bar: "Applied to 12/25 matched jobs"
- Live log feed showing each step

Statistics:
- Total Processed today
- Total Applied today
- High Matches (score > 70) count
- Average match score
- Daily limit remaining

---

## APPLICATIONS PAGE REQUIREMENTS

Table of all applications:
- Job Title, Company, Location
- Match Score (color coded: green/yellow/red)
- Resume Version Used (link to download tailored PDF)
- Applied Date
- Status (Applied / Interview / Rejected / No Response)

Filters: By date, match score, company, status
Export to CSV button

---

## AI PROMPTS

### Job Matching Prompt:
You are a job matching AI assistant. Analyze the compatibility between a candidate and a job posting.

CANDIDATE RESUME:
{resume_text}

JOB POSTING:
Title: {job_title}
Company: {company}
Description: {job_description}
Required Skills: {required_skills}
Experience Required: {experience_range}

Evaluate and return a JSON response:
{
  "match_score": <0-100>,
  "matching_skills": ["skill1", "skill2"],
  "missing_skills": ["skill3"],
  "experience_fit": "good/partial/poor",
  "overall_assessment": "brief 1-2 sentence summary",
  "should_apply": true/false
}

Scoring criteria:
- Skills overlap: 40% weight
- Experience level fit: 25% weight
- Role relevance: 20% weight
- Location/salary fit: 15% weight

### Resume Tailoring Prompt:
You are an expert resume writer. Tailor the following resume for a specific job posting.

ORIGINAL RESUME:
{resume_text}

TARGET JOB:
Title: {job_title}
Company: {company}
Description: {job_description}
Key Requirements: {requirements}

INSTRUCTIONS:
1. Rewrite the Professional Summary to directly address this role
2. Reorder skills — put the most relevant skills for THIS job first
3. For each work experience entry, emphasize achievements relevant to THIS job
4. Add keywords from the job description naturally (for ATS systems)
5. Keep all information TRUTHFUL — do not invent experience or skills
6. Maintain professional formatting

Return a JSON response with sections: professional_summary, skills, experience, education, projects

### Cover Letter Prompt:
Write a brief, professional cover letter for this job application.
CANDIDATE: {candidate_name}
RESUME SUMMARY: {resume_summary}
JOB: {job_title} at {company}
JOB DESCRIPTION: {job_description}
Keep it under 200 words. Be specific about why this candidate fits THIS role.
Tone: Professional but warm. Not robotic.

---

## ANTI-DETECTION SYSTEM

DELAYS:
- Between typing each character: random 50-150ms
- Between filling each form field: random 1-3 seconds
- Between clicking buttons: random 2-5 seconds
- Between submitting applications: random 30-90 seconds
- Break after every 5-10 applications: random 2-5 minutes
- If 20+ applications done: take a 15-30 minute break

HUMAN BEHAVIOR SIMULATION:
- Random mouse movements before clicking
- Occasionally scroll up and down before acting
- Don't always fill forms in the same order
- Sometimes hover over elements before clicking
- Vary typing speed (some words faster, some slower)

LIMITS:
- Max 30 applications per day (configurable)
- Max 10 applications per hour
- Don't apply to the same company twice in one day
- Don't apply to jobs already applied to (check database)

BROWSER:
- Use real browser (not headless) — Playwright headful mode
- Random window size
- Real user agent string
- Don't block images or CSS

---

## DATABASE SCHEMA

CREATE TABLE settings (
  id INTEGER PRIMARY KEY,
  naukri_email TEXT NOT NULL,
  naukri_password TEXT NOT NULL,
  gemini_api_key TEXT NOT NULL,
  job_titles TEXT NOT NULL,
  locations TEXT NOT NULL,
  skills TEXT,
  min_experience INTEGER DEFAULT 0,
  max_experience INTEGER DEFAULT 3,
  min_salary INTEGER,
  match_threshold INTEGER DEFAULT 70,
  max_daily_applications INTEGER DEFAULT 30,
  dry_run BOOLEAN DEFAULT true,
  auto_cover_letter BOOLEAN DEFAULT true,
  master_resume_path TEXT,
  master_resume_text TEXT,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE jobs (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  naukri_job_id TEXT UNIQUE,
  title TEXT NOT NULL,
  company TEXT NOT NULL,
  location TEXT,
  salary TEXT,
  experience TEXT,
  description TEXT,
  required_skills TEXT,
  match_score INTEGER,
  match_reasons TEXT,
  scanned_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE applications (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  job_id INTEGER REFERENCES jobs(id),
  tailored_resume_path TEXT,
  cover_letter TEXT,
  match_score INTEGER,
  status TEXT DEFAULT 'applied',
  applied_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  notes TEXT
);

CREATE TABLE daily_stats (
  date TEXT PRIMARY KEY,
  jobs_scanned INTEGER DEFAULT 0,
  jobs_matched INTEGER DEFAULT 0,
  applications_sent INTEGER DEFAULT 0,
  avg_match_score REAL
);

---

## MONETIZATION

Phase 1 (Launch): Completely free, get 1000 users
Phase 2 (Month 2-3): Freemium
- FREE: 3 apps/day, basic matching, no AI tailoring
- PRO ₹499/month: 30 apps/day, AI resume tailoring, cover letters, analytics
- PREMIUM ₹999/month: Unlimited, multi-platform, interview prep

Phase 3 (Month 6+): B2B — companies pay for access to candidate pool

---

## BUILD ORDER

WEEK 1: Configuration page with form UI, Playwright login to Naukri, job search and extraction, Gemini AI matching, save to database

WEEK 2: Gemini resume tailoring, PDF generation, auto-fill Naukri forms, upload tailored resume, anti-detection, application tracking

WEEK 3: Real-time dashboard progress, applications history page, statistics/charts, dry run testing, error handling

WEEK 4: Landing page, Razorpay payments, user auth, deploy, start marketing

---

## HOW TO CONTINUE BUILDING

Open terminal in project folder and run: claude

Then paste:
"Read the JOBPILOT-VISION.md file first. This is the complete project plan. Then look at the existing codebase. Tell me what's already built and what's missing. Then start building the missing pieces following the BUILD ORDER. Start with the Configuration page and Playwright Naukri login."

---

## KEY PRINCIPLES
1. User experience first — everything from the web UI, no terminal/secrets
2. AI quality matters — bad resume tailoring = bad product
3. Anti-detection is critical — if users get banned, product is dead
4. Start with Naukri only — nail one platform first
5. Dry Run mode always — never auto-submit without testing
6. Track everything — data is your moat for B2B pivot
```

---

**Now:**
1. Create `JOBPILOT-VISION.md` in your Replit project root
2. Paste everything above into it
3. When your credits reset (14 hours) OR in Claude Code, paste:
```
Read the JOBPILOT-VISION.md file first. Then look at the existing code. Tell me what's built and what's missing. Start building the missing pieces.