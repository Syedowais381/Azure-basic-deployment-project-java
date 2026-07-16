# Azure Deployment Guide (Simple)

Follow these steps in the **Azure Portal** (https://portal.azure.com).

---

## Part 1 – Create Azure Resources

### Step 1: Resource Group

1. Search **Resource groups** → **Create**
2. Name: `employee-project-rg`
3. Region: pick one near you (e.g. Central India)
4. **Review + create**

### Step 2: SQL Server

1. Search **SQL servers** → **Create**
2. Resource group: `employee-project-rg`
3. Server name: `employee-sql-server` + your initials (must be globally unique, e.g. `employee-sql-krut`)
4. Authentication: **Use SQL authentication**
5. Set admin login + password (save these!)
6. **Review + create**

### Step 3: SQL Database

1. After server is created, go to it → **Create database**
2. Database name: `EmployeeDB`
3. Compute: **Basic** or **Serverless** (cheapest)
4. **Create**

### Step 4: Firewall (allow your PC + Azure)

1. Open your SQL Server → **Networking** (or Firewalls and virtual networks)
2. **Add your client IPv4 address**
3. Enable **Allow Azure services and resources to access this server**
4. **Save**

### Step 5: App Service Plan

1. Search **App Service plans** → **Create**
2. Resource group: `employee-project-rg`
3. Name: `employee-plan`
4. OS: **Linux**
5. Region: same as resource group
6. Pricing: **Free F1** (Dev/Test tab) — if unavailable in your region, use **Basic B1**
7. **Review + create**

### Step 6: App Service (Web App)

1. Search **Web App** → **Create**
2. Resource group: `employee-project-rg`
3. Name: `employee-app-yourname` (becomes `https://employee-app-yourname.azurewebsites.net`)
4. Publish: **Code**
5. Runtime stack: **Java 21**
6. OS: **Linux**
7. Region: same as above
8. App Service plan: `employee-plan`
9. **Review + create**

---

## Part 2 – Push Code to GitHub

1. Create a new repo on GitHub (e.g. `employee-management`)
2. In your project folder:

```bash
git init
git add .
git commit -m "Initial commit - Employee Management System"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/employee-management.git
git push -u origin main
```

---

## Part 3 – Connect GitHub to Azure

1. Open your **App Service** in Azure Portal
2. Left menu → **Deployment Center**
3. Source: **GitHub**
4. Sign in and select your repo + branch (`main`)
5. Build provider: **GitHub Actions** (Azure creates the workflow for you)
6. **Save** — first deployment starts automatically

Wait 5–10 minutes for the build to finish.

---

## Part 4 – Configure Database Connection

1. App Service → **Configuration** → **Application settings**
2. Add these (click **+ New application setting** for each):

| Name | Value |
|------|-------|
| `SPRING_PROFILES_ACTIVE` | `azure` |
| `AZURE_SQL_USERNAME` | your SQL admin username |
| `AZURE_SQL_PASSWORD` | your SQL admin password |
| `AZURE_SQL_URL` | see below |

**AZURE_SQL_URL** format:

```
jdbc:sqlserver://YOUR-SERVER-NAME.database.windows.net:1433;database=EmployeeDB;encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=30;
```

Replace `YOUR-SERVER-NAME` with your actual SQL server name.

Example:
```
jdbc:sqlserver://employee-sql-krut.database.windows.net:1433;database=EmployeeDB;encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=30;
```

3. Click **Save** → **Continue** (app restarts)

---

## Part 5 – Verify

1. Open `https://YOUR-APP-NAME.azurewebsites.net`
2. Add a few employees
3. Refresh — data should persist (Azure SQL)

---

## Screenshots for assignment

Take screenshots of:

- Resource group with all resources
- SQL Database overview
- App Service overview
- App Service → Configuration (hide password)
- Your live app running in browser
- Deployment Center showing successful deployment

---

## Troubleshooting

**App won't start:** App Service → **Log stream** — check for errors.

**Database connection failed:** Double-check firewall, username/password, and JDBC URL.

**Free F1 not available:** Use Basic B1 (~$13/month) or try another region.
