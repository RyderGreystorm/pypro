# Documentum Active User Report

## Description

A fully automated reporting system that connects to an Oracle database, runs a predefined query on the **first and last Monday of every month**, stores the results in a CSV file, and sends it as an email attachment to company stakeholders. The scheduler also supports running tasks daily or weekly, providing flexible execution for different use cases.

---

## Project Structure

```
user_report_automation/
├── 1.jpg                  # Company logo for email branding
├── README.md              # Project documentation
├── config.py              # Configuration loading from .env files
├── cronjob.py             # Scheduling logic using APScheduler
├── database_ops.py        # Oracle DB operations (connect, query, write CSV)
├── email_ops.py           # Email composition and delivery
├── html_body.py           # HTML email template with embedded logo
├── main.py                # Main entry point that ties everything together
├── report.csv             # Output file generated from the DB query
├── requirements.txt       # Project dependencies
├── .env.secret            # Private credentials (not committed to version control)
└── .env.shared            # Shared configurations
```

---

## Features

- Cron-based scheduling using `BlockingScheduler` from `apscheduler`
- Run tasks on:
  - First and last Monday of each month
  - Weekly (customizable weekday)
  - Daily (customizable time)
- Exports query result to `report.csv`
- Sends styled HTML email with embedded company logo and attached CSV
- Secure credentials and secrets management via `.env.secret` and `.env.shared`
- Block-thread model ensures long-running scheduling process

---

## Installation

1. Clone the repository

   ```bash
   git clone https://github.com/yourusername/user_report_automation.git
   cd user_report_automation
   ```

2. Create a virtual environment and activate it:

   ```bash
   python -m venv .venv
   source .venv/bin/activate      # Linux/macOS
   .venv\Scripts\activate         # Windows
   ```

3. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```

4. Create `.env.secret` and `.env.shared` with appropriate values (see below).

5. Run the script:

   ```bash
   python main.py
   ```

---

## Environment Variables

### .env.secret

```
EMAIL_PASSWORD=your_gmail_app_password
SMTP_SERVER=smtp.gmail.com
SMTP_PORT=465
DB_USER=your_db_user
DB_PASSWORD=your_db_password
DSN=localhost/XEPDB1
SPLUNK_HEC_URL=https://your-splunk-endpoint
SPLUNK_API_KEY=your_splunk_api_key
```

### .env.shared

```
EMAIL=you@company.com
```

---

## Usage

Modify `main.py` to configure the schedule you need:

```python
scheduler = MonthlyScheduler()
scheduler.run_daily(ignition, hour=16, minute=50)     # Run daily
# scheduler.run_weekly(ignition, weekday='wed', hour=9, minute=0)
# scheduler.add_monthly_task(ignition, hour=6, minute=0)
scheduler.start()
```

All tasks should be scheduled before `scheduler.start()` is called since it blocks the thread.

---

## Scheduling Options

| Method               | Description                               |
| -------------------- | ----------------------------------------- |
| `run_daily()`        | Run task daily at specified hour/minute   |
| `run_weekly()`       | Run weekly on a specific weekday and time |
| `add_monthly_task()` | Run on 1st and last Monday of each month  |

---

## Email Report

Each email sent:

- Uses your `EMAIL` address as sender
- Sends `report.csv` as an attachment
- Uses `html_body.py` content for formatting (includes inline logo `1.jpg`)

### Example HTML Snippet:

```html
<img src="cid:companylogo" width="250" alt="Company Logo"/>
<h2>Monthly Users Report</h2>
<p>The attached file contains the summary of Users for the past month.</p>
```

---

## Troubleshooting

- **SMTP Auth Error**: Ensure you use a Gmail App Password (not your regular password).
- **Oracle Connection Fails**: Check `DSN`, `DB_USER`, and `DB_PASSWORD`
- **Scheduler Doesn't Stop**: Use PowerShell or close Git Bash if `Ctrl+C` is not working.
- **Env Not Activated**: Make sure your venv is activated before running the script.

---

## Testing

You can test locally using an Oracle Express instance in Docker or on bare metal:

```bash
docker run -d -p 1521:1521 -e ORACLE_PWD=oracle gvenzl/oracle-xe
```

Then connect via:

- DSN: `localhost/XEPDB1`
- User: `system`
- Password: `oracle`

Make sure port 1521 is not blocked by firewall.

---

## Extending the Project

- Add support for other DBs like PostgreSQL/MySQL
- Push results to Splunk via `SPLUNK_HEC_URL`
- Auto-archive old reports
- Support multiple queries or dynamic query loading

---

## License

This project is intended for internal automation use and follows all S&P Global internal guidelines.

---

## Acknowledgements

- [APScheduler Docs](https://apscheduler.readthedocs.io/)
- Oracle XE Docker Image by [gvenzl](https://github.com/gvenzl)
- Email automation via Python's `smtplib` and `email` libs