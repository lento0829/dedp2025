# Azure Article CMS - Complete Deployment Guide

## ✅ Code Changes Completed

All necessary code changes have been implemented:

### 1. Microsoft Authentication (MSAL) - [FlaskWebProject/views.py](FlaskWebProject/views.py)
- ✅ Implemented `_build_msal_app()` - Creates ConfidentialClientApplication
- ✅ Implemented `_build_auth_url()` - Builds authorization request URL
- ✅ Implemented `_load_cache()` - Loads token cache from session
- ✅ Implemented `_save_cache()` - Saves token cache to session
- ✅ Implemented token acquisition in `authorized()` route
- ✅ Added logging for successful/failed login attempts

### 2. Application Logging - [FlaskWebProject/__init__.py](FlaskWebProject/__init__.py)
- ✅ Configured StreamHandler for logging
- ✅ Set logging level to INFO
- ✅ Added application startup logging

### 3. Deployment Analysis - [WRITEUP.md](WRITEUP.md)
- ✅ Comprehensive comparison of VM vs App Service
- ✅ Justified choice of App Service
- ✅ Detailed scenarios that would change the decision

---

## 🚀 Azure Deployment Steps

### Step 1: Create Azure Resource Group

```bash
# Login to Azure
az login

# Create resource group (replace with your preferred name and location)
az group create --name article-cms-rg --location eastus
```

Or via Azure Portal:
1. Navigate to **Resource Groups**
2. Click **+ Create**
3. Enter name: `article-cms-rg`
4. Select region: `East US` (or your preferred region)
5. Click **Review + create**

---

### Step 2: Create Azure SQL Database

#### Via Azure CLI:
```bash
# Create SQL Server
az sql server create \
  --name article-cms-sql-server \
  --resource-group article-cms-rg \
  --location eastus \
  --admin-user sqladmin \
  --admin-password <YourStrongPassword123!>

# Configure firewall to allow Azure services
az sql server firewall-rule create \
  --resource-group article-cms-rg \
  --server article-cms-sql-server \
  --name AllowAzureServices \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0

# Create database
az sql db create \
  --resource-group article-cms-rg \
  --server article-cms-sql-server \
  --name article-cms-db \
  --service-objective S0
```

#### Via Azure Portal:
1. Navigate to **SQL databases** → Click **+ Create**
2. **Basics:**
   - Resource group: `article-cms-rg`
   - Database name: `article-cms-db`
   - Server: Click **Create new**
     - Server name: `article-cms-sql-server`
     - Admin login: `sqladmin`
     - Password: (strong password)
     - Location: Same as resource group
   - Compute + storage: `Basic` or `Standard S0`
3. **Networking:**
   - Allow Azure services: **Yes**
   - Add current client IP: **Yes**
4. Click **Review + create**

#### Initialize Database Tables:

After creating the database, connect and run the SQL scripts:

**Option 1 - Using Azure Portal Query Editor:**
1. Go to your SQL Database → **Query editor**
2. Login with SQL authentication
3. Run [sql_scripts/users-table-init.sql](sql_scripts/users-table-init.sql)
4. Run [sql_scripts/posts-table-init.sql](sql_scripts/posts-table-init.sql)

**Option 2 - Using Azure Data Studio or SQL Server Management Studio:**
```
Server: article-cms-sql-server.database.windows.net
Database: article-cms-db
Authentication: SQL Login
Username: sqladmin
Password: <your password>
```

**📸 SCREENSHOT REQUIRED:** After running the scripts, take a screenshot showing:
- The created tables (users and posts)
- A SELECT query showing data in both tables

---

### Step 3: Create Azure Blob Storage

#### Via Azure CLI:
```bash
# Create storage account (name must be globally unique, lowercase, no special chars)
az storage account create \
  --name articlecmsstorage123 \
  --resource-group article-cms-rg \
  --location eastus \
  --sku Standard_LRS

# Get storage account key
az storage account keys list \
  --resource-group article-cms-rg \
  --account-name articlecmsstorage123 \
  --query "[0].value" -o tsv

# Create container for images
az storage container create \
  --name images \
  --account-name articlecmsstorage123 \
  --public-access blob
```

#### Via Azure Portal:
1. Navigate to **Storage accounts** → Click **+ Create**
2. **Basics:**
   - Resource group: `article-cms-rg`
   - Storage account name: `articlecmsstorage123` (must be unique)
   - Region: Same as resource group
   - Performance: **Standard**
   - Redundancy: **LRS**
3. Click **Review + create**
4. After creation, go to the storage account
5. Navigate to **Containers** → Click **+ Container**
6. Name: `images`
7. Public access level: **Blob**

**📸 SCREENSHOT REQUIRED:** Take a screenshot showing the blob endpoint URL:
- Go to Storage Account → **Properties** or **Endpoints**
- Show the Blob service endpoint URL

---

### Step 4: Register Application in Azure Active Directory

1. Go to **Azure Active Directory** → **App registrations** → **+ New registration**

2. **Register an application:**
   - Name: `Article-CMS-App`
   - Supported account types: **Accounts in any organizational directory and personal Microsoft accounts**
   - Redirect URI: 
     - Platform: **Web**
     - URI: `https://article-cms-app.azurewebsites.net/getAToken` (adjust domain based on your app name)
   - Click **Register**

3. **Note the Application (client) ID** - You'll need this for config

4. **Create a client secret:**
   - Go to **Certificates & secrets**
   - Click **+ New client secret**
   - Description: `Article-CMS-Secret`
   - Expires: Choose duration (6 months, 12 months, etc.)
   - Click **Add**
   - **IMPORTANT:** Copy the secret **Value** immediately (you won't see it again)

5. **Configure Redirect URIs:**
   - For local testing, add: `http://localhost:5000/getAToken`
   - For Azure App Service: `https://YOUR-APP-NAME.azurewebsites.net/getAToken`

**📸 SCREENSHOT REQUIRED:** Take a screenshot of the Redirect URIs page showing all configured URIs

---

### Step 5: Update Configuration File

Update [config.py](config.py) with your Azure resource values:

```python
# Storage Account Configuration
BLOB_ACCOUNT = 'articlecmsstorage123'
BLOB_STORAGE_KEY = '<your-storage-account-key>'
BLOB_CONTAINER = 'images'

# SQL Database Configuration
SQL_SERVER = 'article-cms-sql-server'
SQL_DATABASE = 'article-cms-db'
SQL_USER_NAME = 'sqladmin'
SQL_PASSWORD = '<your-sql-password>'

# Azure AD Configuration
CLIENT_ID = '<your-application-client-id>'
CLIENT_SECRET = '<your-client-secret-value>'
```

**IMPORTANT:** For production, use environment variables instead of hardcoding credentials!

---

### Step 6: Deploy to Azure App Service

#### Method 1: Using Azure CLI

```bash
# Create App Service Plan
az appservice plan create \
  --name article-cms-plan \
  --resource-group article-cms-rg \
  --sku B1 \
  --is-linux

# Create Web App
az webapp create \
  --resource-group article-cms-rg \
  --plan article-cms-plan \
  --name article-cms-app \
  --runtime "PYTHON:3.10"

# Configure deployment from local git
az webapp deployment source config-local-git \
  --name article-cms-app \
  --resource-group article-cms-rg

# Get deployment credentials
az webapp deployment list-publishing-credentials \
  --name article-cms-app \
  --resource-group article-cms-rg

# Deploy via Git
git remote add azure <deployment-git-url>
git push azure main:master
```

#### Method 2: Using Azure Portal + GitHub

1. **Create App Service:**
   - Navigate to **App Services** → Click **+ Create**
   - **Basics:**
     - Resource Group: `article-cms-rg`
     - Name: `article-cms-app` (must be globally unique)
     - Publish: **Code**
     - Runtime stack: **Python 3.10**
     - Region: Same as resource group
     - Pricing plan: **Basic B1** or higher
   - Click **Review + create**

2. **Configure Application Settings (Environment Variables):**
   - Go to your App Service → **Configuration** → **Application settings**
   - Click **+ New application setting** for each:
     ```
     BLOB_ACCOUNT = articlecmsstorage123
     BLOB_STORAGE_KEY = <your-key>
     BLOB_CONTAINER = images
     SQL_SERVER = article-cms-sql-server
     SQL_DATABASE = article-cms-db
     SQL_USER_NAME = sqladmin
     SQL_PASSWORD = <your-password>
     CLIENT_ID = <your-client-id>
     CLIENT_SECRET = <your-client-secret>
     SECRET_KEY = <generate-random-key>
     ```
   - Click **Save**

3. **Deploy Code:**
   
   **Option A - GitHub Actions (Recommended):**
   - In App Service, go to **Deployment Center**
   - Source: **GitHub**
   - Authorize and select your repository
   - Build Provider: **GitHub Actions**
   - Click **Save**
   - GitHub Actions workflow will be created automatically

   **Option B - ZIP Deployment:**
   ```bash
   # Package the app
   zip -r app.zip . -x "*.git*" ".venv/*" "__pycache__/*"
   
   # Deploy
   az webapp deployment source config-zip \
     --resource-group article-cms-rg \
     --name article-cms-app \
     --src app.zip
   ```

4. **Configure Startup Command:**
   - Go to **Configuration** → **General settings**
   - Startup Command: `gunicorn --bind=0.0.0.0 --timeout 600 application:app`
   - Click **Save**

#### Method 3: Using VS Code Azure Extension

1. Install **Azure App Service** extension
2. Sign in to Azure
3. Right-click on [application.py](application.py) → **Deploy to Web App**
4. Follow prompts to select/create resources

---

### Step 7: Test the Application

1. **Access the application:**
   - URL: `https://YOUR-APP-NAME.azurewebsites.net`

2. **Test Standard Login:**
   - Username: `admin`
   - Password: `pass`
   - Check logs for successful login message

3. **Test Invalid Login:**
   - Try wrong credentials
   - Check logs for invalid login attempt message

4. **Test Microsoft Sign In:**
   - Click "Sign in with Microsoft" button
   - Authenticate with Microsoft account
   - Should redirect back and log in as admin

5. **Create Test Article:**
   - Click **Create** button
   - Fill in:
     - **Title:** "Hello World!"
     - **Author:** "Jane Doe"
     - **Body:** "My name is Jane Doe and this is my first article!"
   - Upload an image (.png or .jpg)
   - Click **Save**
   - Navigate back to the article
   - **📸 SCREENSHOT REQUIRED:** Capture the article page with URL visible

---

### Step 8: Collect Required Screenshots

You need to submit the following screenshots:

1. **✅ Article Created Successfully**
   - Show the "Hello World!" article with URL visible
   - Must include title, author, body, and uploaded image

2. **✅ Resource Group with All Resources**
   - Azure Portal → Resource Groups → article-cms-rg
   - Show all resources created (Storage Account, SQL Server, SQL Database, App Service Plan, App Service)

3. **✅ SQL Database Tables and Query**
   - Query Editor showing users and posts tables
   - SELECT query results showing sample data

4. **✅ Blob Storage Endpoint**
   - Storage Account → Properties or Endpoints
   - Show the blob service endpoint URL

5. **✅ Azure AD Redirect URIs**
   - App Registration → Authentication
   - Show all configured Redirect URIs

6. **✅ Application Logs**
   - App Service → Log stream (or your logging solution)
   - Show both:
     - "Invalid login attempt" message
     - "admin logged in successfully" message

---

### Step 9: View Application Logs

#### Option 1: Azure Portal Log Stream
1. Go to App Service → **Log stream**
2. Wait for connection
3. Perform login actions to generate logs
4. **📸 SCREENSHOT REQUIRED:** Capture invalid and successful login attempts

#### Option 2: Application Insights (Optional)
1. Enable Application Insights for your App Service
2. View logs in **Logs** or **Transaction search**

#### Option 3: Download Logs
```bash
az webapp log download \
  --resource-group article-cms-rg \
  --name article-cms-app \
  --log-file logs.zip
```

---

## 🧹 Cleanup Resources (IMPORTANT!)

**After receiving your grade**, delete all resources to avoid charges:

```bash
# Delete entire resource group (deletes all resources within it)
az group delete --name article-cms-rg --yes --no-wait
```

Or via Azure Portal:
1. Go to **Resource Groups**
2. Select `article-cms-rg`
3. Click **Delete resource group**
4. Type the resource group name to confirm
5. Click **Delete**

---

## 📋 Submission Checklist

- [ ] Code files: [FlaskWebProject/\_\_init\_\_.py](FlaskWebProject/__init__.py) and [FlaskWebProject/views.py](FlaskWebProject/views.py)
- [ ] Analysis: [WRITEUP.md](WRITEUP.md)
- [ ] Screenshot: Article with "Hello World!" and URL
- [ ] Screenshot: Resource group with all resources
- [ ] Screenshot: SQL tables and query results
- [ ] Screenshot: Blob storage endpoints
- [ ] Screenshot: Azure AD redirect URIs
- [ ] Screenshot: Logs showing invalid and successful login
- [ ] Optional: URL to deployed App Service

---

## 🔧 Troubleshooting

### Issue: Application won't start
- Check Application Settings are configured
- Verify startup command: `gunicorn --bind=0.0.0.0 --timeout 600 application:app`
- Check logs in Log stream

### Issue: Database connection fails
- Verify SQL Server firewall allows Azure services
- Check connection string format
- Ensure database credentials are correct

### Issue: Images not displaying
- Verify blob container is set to public access level "Blob"
- Check storage account key is correct
- Confirm container name is "images"

### Issue: Microsoft Sign In doesn't work
- Verify redirect URIs match exactly (including https)
- Check CLIENT_ID and CLIENT_SECRET are correct
- Ensure Azure AD app registration is complete

### Issue: Logs not showing
- Restart the App Service
- Check log stream is connected
- Verify logging is configured in \_\_init\_\_.py

---

## 📚 Additional Resources

- [Azure App Service Documentation](https://docs.microsoft.com/en-us/azure/app-service/)
- [Azure SQL Database Documentation](https://docs.microsoft.com/en-us/azure/azure-sql/)
- [Azure Blob Storage Documentation](https://docs.microsoft.com/en-us/azure/storage/blobs/)
- [MSAL Python Documentation](https://github.com/AzureAD/microsoft-authentication-library-for-python)
- [Flask Documentation](https://flask.palletsprojects.com/)

---

Good luck with your project! 🎉
