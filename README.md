📝 Automated Lead Capture & CRM Pipeline
<div align="center">
Show Image
Show Image
Show Image
Show Image

Real-time lead capture automation from Google Forms to PostgreSQL with instant notifications

Features • Installation • Usage • Configuration

</div>
📋 Overview
An n8n workflow that automatically captures leads from Google Forms, validates them against your database to prevent duplicates, stores them in PostgreSQL, and sends instant email notifications when new leads arrive. Perfect for sales teams, marketing professionals, and small businesses looking to streamline their lead management process.

This workflow eliminates manual data entry, ensures data quality through validation, and provides real-time visibility into new lead acquisition.

✨ Features
📊 Automated Lead Collection
Real-time monitoring of Google Forms submissions
Polls every minute for new responses
Automatic data extraction and formatting
No manual intervention required
🔍 Duplicate Detection
Email-based duplicate checking
SQL query validation before insertion
Prevents redundant database entries
Maintains data integrity
🗄️ PostgreSQL Integration
Structured lead storage
Automatic timestamp tracking (created_at, updated_at)
Lead scoring and status management
Scalable database architecture
📧 Instant Notifications
Gmail integration for real-time alerts
Detailed lead information in email body
Automatic subject line with lead name
Includes database-generated Lead ID
🧹 Data Processing
Automatic data cleaning and trimming
Email normalization (lowercase)
Field validation and formatting
Default values for missing data
🏗️ Architecture
Workflow Diagram
mermaid
graph TD
    A[Google Forms Submission] --> B[Google Sheets Trigger]
    B --> C[Data Processing & Cleaning]
    C --> D[Check for Duplicate Email]
    D --> E{Email Exists?}
    E -->|No| F[Insert into PostgreSQL]
    E -->|Yes| G[End - Skip Duplicate]
    F --> H[Send Gmail Notification]
    H --> I[Complete]
Data Flow
Input: Google Form submission → Google Sheets
Trigger: n8n polls sheet every minute for new rows
Processing: JavaScript code cleans and formats data
Validation: SQL query checks for existing email
Storage: New leads inserted into PostgreSQL
Notification: Gmail alert sent with lead details
🛠️ Tech Stack
Technology	Purpose
n8n	Workflow automation platform
Google Forms	Lead capture interface
Google Sheets API	Form response storage
PostgreSQL	Contact database
Gmail API	Email notifications
JavaScript	Data transformation
📦 Prerequisites
Before you begin, ensure you have:

✅ n8n instance (self-hosted or cloud)
✅ Google Forms with lead capture form
✅ Google Sheets connected to form responses
✅ PostgreSQL database with contacts table
✅ Gmail account with API access
✅ Google Cloud project with required APIs enabled
Database Schema
Create the contacts table in PostgreSQL:

sql
CREATE TABLE contacts (
    id SERIAL PRIMARY KEY,
    first_name VARCHAR(255),
    last_name VARCHAR(255),
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(50),
    company VARCHAR(255),
    lead_source VARCHAR(255),
    lead_status VARCHAR(50) DEFAULT 'new',
    lead_score INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Index for faster email lookups
CREATE INDEX idx_email ON contacts(email);
🚀 Installation
Step 1: Clone the Repository
bash
git clone https://github.com/yourusername/lead-capture-automation.git
cd lead-capture-automation
Step 2: Import Workflow to n8n
Open your n8n instance
Click "Import from File" or "Import from URL"
Select workflow.json from this repository
The workflow will be imported with all nodes
Step 3: Set Up Google Form
Create a Google Form with these fields:

First Name (Short answer)
Last Name (Short answer)
Email (Email field)
Phone Number (Short answer)
Company (Short answer)
How did you hear about us? (Dropdown/Multiple choice)
Step 4: Connect Form to Google Sheets
In Google Forms, go to Responses tab
Click the Google Sheets icon
Create a new spreadsheet named "Lead Capture Form (Responses)"
Note the Sheet ID from the URL
Step 5: Configure n8n Credentials
Google Sheets Trigger
Type: OAuth2
Follow n8n's Google Sheets authentication flow
Grant permissions: Read access to sheets
PostgreSQL
Host: your-database-host.com
Database: your_database_name
User: your_db_user
Password: your_db_password
Port: 5432 (default)
SSL: Enable if required
Gmail
Type: OAuth2
Follow n8n's Gmail authentication flow
Scopes: gmail.send
Step 6: Update Workflow Settings
Google Sheets Trigger Node
Update documentId with your Google Sheets URL
Set sheetName to "Form Responses 1"
Verify poll time (every minute)
Gmail Node
Update sendTo with your notification email
Customize subject line if needed
⚙️ Configuration
Form Field Mapping
The workflow expects these Google Form field names:

Form Field	Database Column	Processing
First Name	first_name	Trimmed
Last Name	last_name	Trimmed
Email	email	Lowercase, trimmed
Phone Number	phone	Trimmed
Company	company	Trimmed
How did you hear about us?	lead_source	Trimmed, default: "Unknown"
Data Processing Logic
javascript
// Automatic data cleaning in Code node
return items.map(item => ({
  first_name: item.json['First Name']?.toString().trim() || '',
  last_name: item.json['Last Name']?.toString().trim() || '',
  email: item.json['Email']?.toString().toLowerCase().trim() || '',
  phone: item.json['Phone Number']?.toString().trim() || '',
  company: item.json['Company']?.toString().trim() || '',
  lead_source: item.json['How did you hear about us?']?.toString().trim() || 'Unknown',
  lead_status: 'new',
  lead_score: 0
}));
Duplicate Detection Query
sql
SELECT COUNT(*) as count 
FROM contacts 
WHERE email = '{{ $json.email }}';
If count = 0, lead is inserted. Otherwise, workflow ends.

💡 Usage
Running the Workflow
Activate the Workflow
Click "Active" toggle in n8n
Workflow will poll Google Sheets every minute
Submit a Test Lead
Fill out your Google Form
Wait up to 1 minute for processing
Verify Processing
Check n8n execution logs
Verify database entry: SELECT * FROM contacts ORDER BY created_at DESC LIMIT 1;
Check your Gmail for notification
Workflow Behavior
New Lead (Email Not in Database)
✅ Data extracted from Google Sheets
✅ Email checked against database
✅ Lead inserted into PostgreSQL
✅ Notification email sent
✅ Execution marked as success
Duplicate Lead (Email Already Exists)
✅ Data extracted from Google Sheets
✅ Email checked against database
⚠️ Duplicate detected - insertion skipped
⚠️ No notification sent
✅ Execution ends gracefully
Email Notification Format
Subject: New Lead: [First Name] [Last Name]

Body:
New lead captured!

Name: [First Name] [Last Name]
Email: [Email Address]
Phone: [Phone Number]
Company: [Company Name]
Source: [Lead Source]

Lead ID: [Database Generated ID]
📊 Database Management
View All Leads
sql
SELECT * FROM contacts 
ORDER BY created_at DESC;
Check for Duplicates
sql
SELECT email, COUNT(*) as count 
FROM contacts 
GROUP BY email 
HAVING COUNT(*) > 1;
Update Lead Status
sql
UPDATE contacts 
SET lead_status = 'contacted', 
    updated_at = CURRENT_TIMESTAMP 
WHERE id = [lead_id];
Lead Analytics
sql
-- Leads by source
SELECT lead_source, COUNT(*) as total 
FROM contacts 
GROUP BY lead_source 
ORDER BY total DESC;

-- Recent leads (last 7 days)
SELECT * FROM contacts 
WHERE created_at >= NOW() - INTERVAL '7 days' 
ORDER BY created_at DESC;
🔒 Security Best Practices
✅ Store database credentials in n8n credentials manager
✅ Use environment variables for sensitive data
✅ Enable SSL for PostgreSQL connections
✅ Restrict Gmail API scopes to minimum required
✅ Regularly audit database access logs
✅ Implement row-level security in PostgreSQL if needed
✅ Use service accounts for Google API access
🐛 Troubleshooting
Common Issues
Problem	Solution
No new leads detected	Check Google Sheets Trigger is active and polling interval is correct
Duplicate entries despite check	Verify email field has UNIQUE constraint in database
Gmail not sending	Confirm OAuth2 credentials are valid and have gmail.send scope
Database connection failed	Check PostgreSQL credentials, host, port, and firewall rules
Form fields not mapping	Ensure Google Form field names match exactly (case-sensitive)
Execution errors	Check n8n logs for detailed error messages
Debug Mode
Enable detailed logging in Code node:

javascript
// Add console logging
console.log('Processing item:', item);
console.log('Cleaned data:', result);
return result;
View logs in n8n execution history.

📈 Performance Optimization
Polling Frequency: Adjust from 1 minute to match your lead volume
Database Indexing: Ensure email column has index for fast lookups
Connection Pooling: Enable in PostgreSQL for high-traffic scenarios
Error Handling: Add retry logic for transient failures
Rate Limiting: Monitor Google API quotas
Recommended Settings
javascript
// For high-volume scenarios
pollTimes: {
  item: [{ mode: "everyMinute" }]  // Up to 1440 checks/day
}

// For low-volume scenarios
pollTimes: {
  item: [{ mode: "everyHour" }]  // 24 checks/day, lower API usage
}
🔮 Future Enhancements
 Lead scoring algorithm based on company size/industry
 Integration with CRM systems (HubSpot, Salesforce)
 Automated follow-up email sequences
 Slack/Discord notifications
 Lead assignment to sales reps
 A/B testing for form variations
 Advanced analytics dashboard
 Webhook support for instant triggers
 Multi-language form support
 GDPR compliance automation
🤝 Contributing
Contributions are welcome! Please follow these steps:

Fork the repository
Create a feature branch (git checkout -b feature/AmazingFeature)
Commit your changes (git commit -m 'Add some AmazingFeature')
Push to the branch (git push origin feature/AmazingFeature)
Open a Pull Request
📄 License
This project is licensed under the MIT License - see the LICENSE file for details.

🙏 Acknowledgments
n8n Community - For the powerful automation platform
PostgreSQL Team - For reliable database technology
Google Cloud - For Forms and Sheets integration
📞 Support
For questions or issues:

📧 Email: Zandergarcia552@gmail.com 
🐛 Issues: GitHub Issues
💬 Discussions: GitHub Discussions
<div align="center">
⭐ Star this repo if you find it helpful!

Made with ❤️ for sales and marketing teams

</div>
