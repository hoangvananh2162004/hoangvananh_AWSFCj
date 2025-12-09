---
title: "Visualizing Analytics with Shiny Dashboards"
weight: 55
chapter: false
pre: " <b> 5.5. </b> "
---

## 5.5.1 Environment information

- OS: **Ubuntu 22.04 (Jammy)** – EC2 in a private subnet  
- PostgreSQL: **v18** (installed from the `apt.postgresql.org` repo)  
- Shiny Server: `.deb` binary from RStudio (Posit)  
- User running Shiny: `shiny`  
- App path: `/srv/shiny-server/sbw_dashboard/app.R`

---

## 5.5.2 Install system packages (system libs)

Note: you need to enable the NAT Gateway before downloading system packages.  
Log in to EC2 using **SSM Session Manager** or SSH (temporarily, if available), then run:

```bash
# 1) Update package list
sudo apt-get update

# 2) Install R (if not installed)
sudo apt-get install -y r-base

# 3) Install Postgres client & dev headers (for RPostgres)
#    If your DB is PG 18 then use postgresql-server-dev-18
#    (if your version is different, change 18 -> 14, 15, ...)
sudo apt-get install -y postgresql-client-18 postgresql-server-dev-18

# 4) Install libpq + libssl (required to build RPostgres)
sudo apt-get install -y libpq-dev libssl-dev

# 5) (If Shiny Server is not installed yet)
#    Depending on how you install it, just remember:
#    - shiny-server service: /etc/systemd/system/shiny-server.service
#    - app folder: /srv/shiny-server/
#    - user: shiny
```

Check that `libpq` and dev headers are present:

```bash
dpkg -l | grep -E 'libpq-dev|postgresql-server-dev' || echo "MISSING_LIBS"
ls -l /usr/include/postgresql/libpq-fe.h || echo "NO_LIBPQ_HEADER"
```

If you **do not see any error** → OK.

---

## 5.5.3 Configure R library folder for user `shiny`

To let Shiny Server load R packages, we install the packages under user `shiny` and use the folder:

- `/home/shiny/R/x86_64-pc-linux-gnu-library/4.1`

Run:

```bash
sudo -u shiny R --vanilla <<'EOF'
# Create library directory for user shiny if it does not exist
dir.create(Sys.getenv("R_LIBS_USER"), recursive = TRUE, showWarnings = FALSE)

# Put R_LIBS_USER at the top of .libPaths()
.libPaths(c(Sys.getenv("R_LIBS_USER"), .libPaths()))
cat("LIBPATHS:
"); print(.libPaths())

q("no")
EOF
```

You should see `LIBPATHS` where line 1 is `/home/shiny/R/x86_64-pc-linux-gnu-library/4.1`.

---

## 5.5.4 Install required R packages

Packages needed for the dashboard:

- `shiny`
- `DBI`
- `RPostgres`
- `dplyr`
- `ggplot2`
- `lubridate`
- `pool`

Install all of them under user `shiny`:

```bash
sudo -u shiny R --vanilla <<'EOF'
dir.create(Sys.getenv("R_LIBS_USER"), recursive = TRUE, showWarnings = FALSE)
.libPaths(c(Sys.getenv("R_LIBS_USER"), .libPaths()))
cat("LIBPATHS:
"); print(.libPaths())

install.packages(
  c("shiny", "DBI", "RPostgres", "dplyr", "ggplot2", "lubridate", "pool"),
  repos = "https://cloud.r-project.org"
)

q("no")
EOF
```

💡 **If you hit errors related to `libpq-fe.h` or `libpq`:**

1. Re-check that `libpq-dev`, `postgresql-server-dev-XX`, `libssl-dev` are installed.  
2. Re-run `install.packages("RPostgres", ...)` after installing all required libs.  

Verify that packages can be loaded:

```bash
sudo -u shiny R --vanilla <<'EOF'
.libPaths(c(Sys.getenv("R_LIBS_USER"), .libPaths()))
cat("LIBPATHS:
"); print(.libPaths())

library(shiny)
library(DBI)
library(RPostgres)
library(dplyr)
library(ggplot2)
library(lubridate)
library(pool)

cat("All packages loaded OK
")
q("no")
EOF
```

If there is **no error** → the R environment is OK.

---

## 5.5.5 Deploying the Shiny app

### 5.5.5.1 Create the app folder and copy code

```bash
sudo mkdir -p /srv/shiny-server/sbw_dashboard
sudo chown -R shiny:shiny /srv/shiny-server/sbw_dashboard
```

Create (or replace) the app file:

```bash
sudo nano /srv/shiny-server/sbw_dashboard/app.R
# PASTE THE FULL app.R CODE (the full version you are using)
# Ctrl+O, Enter, Ctrl+X to save
```

Make sure permissions are correct:

```bash
sudo chown shiny:shiny /srv/shiny-server/sbw_dashboard/app.R
sudo chmod 644 /srv/shiny-server/sbw_dashboard/app.R
```

### 5.5.5.2 Restart Shiny Server

```bash
sudo systemctl restart shiny-server
sudo systemctl status shiny-server
```

---

## 5.5.6 Check the app from EC2 (local)

From an SSM session on EC2 (terminal):

```bash
# Check Shiny welcome page
curl -m 5  -sS -o /dev/null -w "WELCOME HTTP %{http_code}
"   http://127.0.0.1:3838/

# Check SBW dashboard app
curl -m 10 -sS -o /dev/null -w "DASHBOARD HTTP %{http_code}
"   http://127.0.0.1:3838/sbw_dashboard/
```

If it returns `DASHBOARD HTTP 200` → the app is running OK.

If it returns `500`:

```bash
LATEST=$(ls -1t /var/log/shiny-server/sbw_dashboard-shiny-*.log | head -n 1)
echo "LATEST=$LATEST"
sudo tail -n 100 "$LATEST"
```

Check the error log for debugging.

---

## 5.5.7 Access the dashboard from your local machine

Because the EC2 instance is in a **private subnet**, you use **SSM port forwarding**:

```bash
# Example using AWS CLI v2 on your local machine:
aws ssm start-session   --target <INSTANCE_ID_PRIVATE>   --document-name AWS-StartPortForwardingSessionToRemoteHost   --parameters '{"host":["127.0.0.1"],"portNumber":["3838"],"localPortNumber":["3838"]}'
```

After that, open your browser on your local machine at:

```text
http://127.0.0.1:3838/sbw_dashboard/
```

The dashboard will show, for example:

- **KPI cards** (total events, users, sessions, …)  
- Charts for **events over time**, **event mix**, **events by login state**  
- **Products & Raw sample** tab (pagination, newest first, auto refresh every 10s – depending on your app code)

---

## 5.5.8 Quick summary of important commands

```bash
# Install system libs
sudo apt-get update
sudo apt-get install -y r-base postgresql-client-18 postgresql-server-dev-18 libpq-dev libssl-dev

# Install R packages for user shiny
sudo -u shiny R --vanilla <<'EOF'
dir.create(Sys.getenv("R_LIBS_USER"), recursive = TRUE, showWarnings = FALSE)
.libPaths(c(Sys.getenv("R_LIBS_USER"), .libPaths()))
install.packages(
  c("shiny", "DBI", "RPostgres", "dplyr", "ggplot2", "lubridate", "pool"),
  repos = "https://cloud.r-project.org"
)
q("no")
EOF

# Deploy app
sudo mkdir -p /srv/shiny-server/sbw_dashboard
sudo nano /srv/shiny-server/sbw_dashboard/app.R   # paste code
sudo chown -R shiny:shiny /srv/shiny-server/sbw_dashboard
sudo systemctl restart shiny-server

# Check dashboard
curl -m 10 -sS -o /dev/null -w "DASHBOARD HTTP %{http_code}
"   http://127.0.0.1:3838/sbw_dashboard/
```
