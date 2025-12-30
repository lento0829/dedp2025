# Azure Article CMS - Quick Reference

## ✅ What Has Been Completed

### 1. Code Implementation ✅

All TODO items in the codebase have been implemented:

#### [FlaskWebProject/views.py](FlaskWebProject/views.py)
- **Line 71-79**: Added logging for invalid login attempts
- **Line 73**: `app.logger.warning('Invalid login attempt for username: {}'.format(form.username.data))`
- **Line 77**: `app.logger.info('User {} logged in successfully'.format(user.username))`
- **Line 91-97**: Implemented token acquisition with MSAL
- **Line 104**: Added logging for Microsoft authentication
- **Line 120-124**: Implemented `_load_cache()` function
- **Line 126-129**: Implemented `_save_cache()` function
- **Line 131-137**: Implemented `_build_msal_app()` function
- **Line 139-145**: Implemented `_build_auth_url()` function

#### [FlaskWebProject/__init__.py](FlaskWebProject/__init__.py)
- **Lines 13-17**: Configured logging with StreamHandler
- Set log level to INFO
- Added application startup log message

#### [WRITEUP.md](WRITEUP.md)
- Comprehensive analysis of VM vs App Service
- Cost, scalability, availability, and workflow comparison
- Justified choice: **App Service**
- Detailed scenarios that would change the decision

---

## 🔑 Key Implementation Details

### Microsoft Authentication (MSAL)
```python
# Builds MSAL Confidential Client
_build_msal_app(cache, authority) 
  → Returns ConfidentialClientApplication with CLIENT_ID, CLIENT_SECRET

# Generates Microsoft login URL
_build_auth_url(authority, scopes, state)
  → Returns authorization request URL with redirect URI

# Token cache management
_load_cache() → Loads SerializableTokenCache from session
_save_cache(cache) → Saves cache to session if changed

# Token acquisition
acquire_token_by_authorization_code()
  → Exchanges auth code for access token
```

### Logging Implementation
```python
# In __init__.py
streamHandler = logging.StreamHandler()
streamHandler.setLevel(logging.INFO)
app.logger.addHandler(streamHandler)
app.logger.setLevel(logging.INFO)

# In views.py - Login attempts
app.logger.warning('Invalid login attempt for username: {}'.format(username))
app.logger.info('User {} logged in successfully'.format(username))
app.logger.info('User {} logged in successfully via Microsoft Authentication'.format(username))
```

---

## 📋 Before Deployment Checklist

### Azure Resources to Create:
- [ ] **Resource Group** (e.g., article-cms-rg)
- [ ] **SQL Server** (e.g., article-cms-sql-server)
- [ ] **SQL Database** (e.g., article-cms-db)
  - [ ] Run users-table-init.sql
  - [ ] Run posts-table-init.sql
- [ ] **Storage Account** (e.g., articlecmsstorage123)
  - [ ] Create "images" container with Blob public access
- [ ] **Azure AD App Registration**
  - [ ] Get Client ID
  - [ ] Generate Client Secret
  - [ ] Configure Redirect URIs
- [ ] **App Service Plan** (B1 or higher)
- [ ] **App Service / Web App**

### Configuration Required:

Update [config.py](config.py) or set Environment Variables:

```python
# Blob Storage
BLOB_ACCOUNT = 'your-storage-account-name'
BLOB_STORAGE_KEY = 'your-storage-key'
BLOB_CONTAINER = 'images'

# SQL Database
SQL_SERVER = 'your-sql-server-name'
SQL_DATABASE = 'your-database-name'
SQL_USER_NAME = 'your-sql-username'
SQL_PASSWORD = 'your-sql-password'

# Azure AD
CLIENT_ID = 'your-application-client-id'
CLIENT_SECRET = 'your-client-secret'
AUTHORITY = 'https://login.microsoftonline.com/common'
REDIRECT_PATH = '/getAToken'

# App
SECRET_KEY = 'generate-a-random-secret-key'
```

---

## 🧪 Testing Checklist

After deployment, test the following:

### Standard Login
1. Navigate to: `https://YOUR-APP.azurewebsites.net`
2. Login with:
   - Username: `admin`
   - Password: `pass`
3. **Check logs**: Should see "User admin logged in successfully"

### Invalid Login
1. Try logging in with wrong credentials
2. **Check logs**: Should see "Invalid login attempt for username: [username]"

### Microsoft Sign In
1. Click "Sign in with Microsoft" button
2. Authenticate with Microsoft account
3. Should redirect back and log in automatically
4. **Check logs**: Should see "User admin logged in successfully via Microsoft Authentication"

### Create Article
1. Click **Create** button
2. Fill in:
   - Title: "Hello World!"
   - Author: "Jane Doe"
   - Body: "My name is Jane Doe and this is my first article!"
3. Upload a .png or .jpg image
4. Click **Save**
5. Verify article displays correctly with image

---

## 📸 Required Screenshots

1. **Article page** showing "Hello World!" article with URL visible
2. **Resource Group** showing all created Azure resources
3. **SQL Query** results from users and posts tables
4. **Blob Storage** endpoint URL
5. **Azure AD** Redirect URIs configuration
6. **Application logs** showing:
   - Invalid login attempt message
   - Successful login message

---

## 🎯 Submission Files

Include these files in your submission:

1. **Code Files:**
   - [FlaskWebProject/__init__.py](FlaskWebProject/__init__.py)
   - [FlaskWebProject/views.py](FlaskWebProject/views.py)
   - [config.py](config.py) (with values filled in or referenced as env vars)
   - Other project files as needed

2. **Documentation:**
   - [WRITEUP.md](WRITEUP.md) (analysis completed)
   - [README.md](README.md) (if you add deployment notes)

3. **Screenshots:** (6 total as listed above)

4. **Optional:** URL to deployed application

---

## 🚨 Important Reminders

### Security Best Practices:
- ⚠️ **DO NOT** commit secrets to Git
- ⚠️ Use environment variables for sensitive data
- ⚠️ Keep CLIENT_SECRET secure
- ⚠️ Use Azure Key Vault for production

### Cost Management:
- 💰 Monitor your Azure spending dashboard
- 💰 Set up cost alerts
- 💰 **DELETE resources immediately after grading** to avoid charges
- 💰 Stop App Service when not actively testing

### Redirect URI Configuration:
- 🔗 Must match exactly (case-sensitive)
- 🔗 Include both http://localhost:5000/getAToken (for testing)
- 🔗 Include https://YOUR-APP.azurewebsites.net/getAToken (for production)
- 🔗 Use `_scheme='https'` in production redirect URIs

---

## 🐛 Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| App won't start | Check Application Settings, verify startup command |
| Database connection fails | Check SQL firewall rules, verify credentials |
| Images not showing | Ensure blob container public access is set to "Blob" |
| MS Login fails | Verify redirect URIs match exactly, check CLIENT_ID/SECRET |
| No logs appearing | Restart app service, check Log Stream connection |

---

## 📞 Help Resources

- **Full Deployment Guide**: See [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md)
- **Azure Documentation**: [docs.microsoft.com/azure](https://docs.microsoft.com/azure)
- **MSAL Python**: [GitHub - MSAL Python](https://github.com/AzureAD/microsoft-authentication-library-for-python)
- **Flask Documentation**: [flask.palletsprojects.com](https://flask.palletsprojects.com/)

---

## ✨ You're Ready!

All code changes are complete. Follow the deployment guide to:
1. Create Azure resources
2. Configure your app
3. Deploy to Azure App Service
4. Test and capture screenshots
5. Submit your project

Good luck! 🎉
