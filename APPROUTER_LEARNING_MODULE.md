# SAP Approuter — Complete Learning Module

> **Author:** Deep | **Level:** Beginner to Intermediate  
> **Prerequisites:** Basic Node.js knowledge, SAP BTP trial account  
> **Project:** deepappRouter (this repository)

---

## Table of Contents

1. [What is Approuter? (The Story)](#chapter-1-what-is-approuter)
2. [Why Do We Need It?](#chapter-2-why-do-we-need-it)
3. [Project Architecture](#chapter-3-project-architecture)
4. [Chapter 4: Setup From Scratch (Step-by-Step)](#chapter-4-setup-from-scratch)
5. [Chapter 5: Understanding xs-app.json (Routing)](#chapter-5-understanding-xs-appjson)
6. [Chapter 6: Understanding Destinations](#chapter-6-understanding-destinations)
7. [Chapter 7: Understanding xs-security.json (Authentication)](#chapter-7-understanding-xs-securityjson)
8. [Chapter 8: Running Locally](#chapter-8-running-locally)
9. [Chapter 9: Cloud Deployment with manifest.yaml](#chapter-9-cloud-deployment-with-manifestyaml)
10. [Chapter 10: Cloud Deployment with mta.yaml](#chapter-10-cloud-deployment-with-mtayaml)
11. [Chapter 11: Common Errors & Fixes](#chapter-11-common-errors--fixes)
12. [Chapter 12: Quick Reference Cheatsheet](#chapter-12-quick-reference-cheatsheet)

---

# Chapter 1: What is Approuter?

## Think of it Like a Security Guard + Receptionist

Imagine a big office building:
- **Security Guard** = Checks your ID card (authentication)
- **Receptionist** = Tells you which floor to go to (routing)

That's exactly what the **SAP Application Router** (`@sap/approuter`) does for your web applications.

```
You (Browser) ──► Security Guard (checks login) ──► Receptionist (which backend?)
                         │                                    │
                    "Show me your                    "Go to Floor 1 for API"
                     BTP login"                      "Go to Floor 2 for CAP"
```

## In Technical Terms

The approuter is a **Node.js application** that:
1. **Authenticates users** using SAP XSUAA (OAuth 2.0)
2. **Routes requests** to different backend services based on URL patterns
3. **Serves static files** (HTML, CSS, JS) from a `webapp/` folder
4. **Manages sessions** and CSRF tokens

```
┌─────────────────────────────────────────────────┐
│               SAP APPROUTER (:5000)             │
│                                                 │
│  Browser ──► /api/*        ──► Express Server   │ (port 4040)
│              /odata/*      ──► CAP Application  │ (port 4004)
│              /animaladoption/* ──► CAP App      │ (port 4004)
│              /*            ──► Static webapp/   │ (local files)
│                                                 │
└─────────────────────────────────────────────────┘
```

---

# Chapter 2: Why Do We Need It?

## Without Approuter (The Problem)

```
Browser ──► http://localhost:4040  (Microservice — anyone can access!)
Browser ──► http://localhost:4004  (CAP App — anyone can access!)
Browser ──► http://my-ui.com      (UI — no authentication!)
```

**Problems:**
- No central login — users hit backends directly
- No security — anyone with the URL can access
- Multiple URLs to manage
- CORS (Cross-Origin) issues between UI and backend

## With Approuter (The Solution)

```
Browser ──► http://localhost:5000  (ONE entry point)
              │
              ├── /api/*           ──► Microservice (protected)
              ├── /odata/*         ──► CAP Backend (protected)
              └── /*               ──► Static UI files
```

**Benefits:**
- **Single URL** for everything
- **Central authentication** (login once, access all)
- **No CORS** issues (same origin)
- **Security** — backends are hidden behind the approuter

---

# Chapter 3: Project Architecture

## Our Project Structure

```
deepappRouter/                       ← Root folder
│
├── approuter/                       ← The Approuter (Chapter 4)
│   ├── package.json                 ← Only dependency: @sap/approuter
│   ├── xs-app.json                  ← Routing rules (Chapter 5)
│   ├── default-env.json             ← Local destinations (Chapter 6)
│   └── webapp/
│       └── index.html               ← Landing page (static file)
│
├── node_micro_service/              ← Backend 1: Express.js API
│   ├── package.json
│   ├── server.js                    ← Simple Express server on port 4040
│   └── webapp/
│       ├── data.json
│       └── index.html
│
├── CAP_annotation_based_Application_local/  ← Backend 2: CAP Application
│   ├── package.json
│   ├── db/schema.cds                ← Database schema
│   ├── srv/service.cds              ← OData service definition
│   ├── srv/service.js               ← Service logic
│   └── app/animaladoption/          ← Fiori Elements UI
│
├── manifest.yaml                    ← Cloud Foundry deployment (Chapter 9)
├── xs-security.json                 ← XSUAA config (Chapter 7)
└── APPROUTER_LEARNING_MODULE.md     ← This file!
```

## How They Connect

```
                    ┌──────────────┐
                    │   Browser    │
                    └──────┬───────┘
                           │
                           ▼
                ┌────────────────────┐
                │   APPROUTER :5000  │
                │                    │
                │  xs-app.json       │  ← "Which URL goes where?"
                │  default-env.json  │  ← "Where are the backends?"
                │  webapp/           │  ← "Static files live here"
                └──┬──────────┬──────┘
                   │          │
          /api/*   │          │  /odata/*, /animaladoption/*
                   ▼          ▼
          ┌──────────┐  ┌──────────┐
          │ Express  │  │   CAP    │
          │  :4040   │  │  :4004   │
          └──────────┘  └──────────┘
```

---

# Chapter 4: Setup From Scratch

## Step 1: Create the Approuter Folder

```bash
mkdir approuter
cd approuter
```

## Step 2: Initialize and Install

```bash
npm init -y
npm install @sap/approuter
```

This creates `package.json`:

```json
{
  "name": "approuter",
  "version": "1.0.0",
  "scripts": {
    "start": "node node_modules/@sap/approuter/approuter.js"
  },
  "dependencies": {
    "@sap/approuter": "^21.3.0"
  }
}
```

### What's happening here?

| Field | Meaning |
|-------|---------|
| `"start"` script | Runs the SAP approuter binary — you don't write any server code! |
| `@sap/approuter` | The official SAP package that does all the magic |

> **Key insight:** Unlike Express where you write `server.js`, the approuter is a **pre-built application**. You only configure it with JSON files.

## Step 3: Create `xs-app.json` (Routing Rules)

```json
{
  "welcomeFile": "/index.html",
  "authenticationMethod": "none",
  "routes": [
    {
      "source": "^/api/(.*)$",
      "target": "/$1",
      "destination": "microservice_api",
      "authenticationType": "none"
    },
    {
      "source": "^(.*)$",
      "target": "$1",
      "localDir": "webapp",
      "authenticationType": "none"
    }
  ]
}
```

## Step 4: Create `default-env.json` (Local Destinations)

```json
{
  "destinations": [
    {
      "name": "microservice_api",
      "url": "http://localhost:4040",
      "forwardAuthToken": false
    }
  ]
}
```

## Step 5: Create `webapp/index.html`

```html
<!DOCTYPE html>
<html>
<body>
  <h1>My Approuter is Working!</h1>
  <a href="/api/">Go to API</a>
</body>
</html>
```

## Step 6: Run It!

```bash
# Terminal 1 — Start your backend
cd node_micro_service
npm start
# Server started at http://localhost:4040

# Terminal 2 — Start the approuter
cd approuter
npm start
# Application router is listening on port: 5000
```

Now open `http://localhost:5000` — you see the landing page!  
Click the API link — it proxies to the Express server!

---

# Chapter 5: Understanding xs-app.json

## This is the BRAIN of the Approuter

`xs-app.json` tells the approuter: **"When a URL matches this pattern, send it THERE."**

## Our Current xs-app.json (Line by Line)

```json
{
  "welcomeFile": "/index.html",
  "authenticationMethod": "none",
  "routes": [
    {
      "source": "^/api/(.*)$",
      "target": "/$1",
      "destination": "microservice_api",
      "authenticationType": "none",
      "csrfProtection": false
    },
    {
      "source": "^/odata/(.*)$",
      "target": "/odata/$1",
      "destination": "cap_api",
      "authenticationType": "none"
    },
    {
      "source": "^/animaladoption/(.*)$",
      "target": "/$1",
      "destination": "cap_api",
      "authenticationType": "none"
    },
    {
      "source": "^/sap/(.*)$",
      "target": "/sap/$1",
      "destination": "cap_api",
      "authenticationType": "none"
    },
    {
      "source": "^(.*)$",
      "target": "$1",
      "localDir": "webapp",
      "authenticationType": "none"
    }
  ]
}
```

## Breaking Down Each Route

### Route 1: Express Microservice API

```json
{
  "source": "^/api/(.*)$",
  "target": "/$1",
  "destination": "microservice_api"
}
```

**In plain English:** "If URL starts with `/api/`, strip the `/api` part and send the rest to the Express server."

```
What you type:     http://localhost:5000/api/data.json
                                         ├─┤ ├───────┤
                                  matched    captured as $1
                                  & removed

What approuter sends:  http://localhost:4040/data.json
                                             ├───────┤
                                             $1 = "data.json"
```

**More examples:**

| You access (port 5000) | Approuter sends to Express (port 4040) |
|------------------------|--------------------------------------|
| `/api/` | `/` |
| `/api/data.json` | `/data.json` |
| `/api/vendor.json` | `/vendor.json` |

### Route 2: CAP OData Service (Direct)

```json
{
  "source": "^/odata/(.*)$",
  "target": "/odata/$1",
  "destination": "cap_api"
}
```

**In plain English:** "If URL starts with `/odata/`, keep the `/odata` part and send to CAP."

```
What you type:        http://localhost:5000/odata/v4/adoptor/Animals
                                            ├───────────────────────┤
                                            captured as $1

What approuter sends: http://localhost:4004/odata/v4/adoptor/Animals
                                            ├───────────────────────┤
                                            target = /odata/ + $1
```

> **Notice:** Here `target` is `/odata/$1` (keeps the prefix). In Route 1 it was `/$1` (strips the prefix). This is how you control URL rewriting!

### Route 3: CAP via Animaladoption Prefix

```json
{
  "source": "^/animaladoption/(.*)$",
  "target": "/$1",
  "destination": "cap_api"
}
```

**In plain English:** "If URL starts with `/animaladoption/`, strip that prefix and send the rest to CAP."

```
What you type:        http://localhost:5000/animaladoption/odata/v4/adoptor/
                                            ├───────────────────────────────┤
                                            captured as $1 = "odata/v4/adoptor/"

What approuter sends: http://localhost:4004/odata/v4/adoptor/
                                            ├───────────────┤
                                            target = / + $1
```

**Why do we need this?** It lets you access the SAME CAP service via two different URL patterns:
- `/odata/v4/adoptor/Animals` — direct
- `/animaladoption/odata/v4/adoptor/Animals` — via prefix

### Route 4: SAP Fiori Flexibility

```json
{
  "source": "^/sap/(.*)$",
  "target": "/sap/$1",
  "destination": "cap_api"
}
```

**Why?** Fiori Elements UI makes internal calls to `/sap/bc/lrep/flex/*` for UI personalization. Without this route, these calls hit the `webapp/` catch-all and fail with 404.

### Route 5: Static Files (CATCH-ALL)

```json
{
  "source": "^(.*)$",
  "target": "$1",
  "localDir": "webapp"
}
```

**In plain English:** "Everything else? Look for it in the `webapp/` folder."

```
http://localhost:5000/index.html  ──► approuter/webapp/index.html
http://localhost:5000/style.css   ──► approuter/webapp/style.css
```

## The Golden Rule: ORDER MATTERS!

Routes are checked **top to bottom**. First match wins.

```
Request: /api/data.json

Check Route 1: ^/api/(.*)$          ──► MATCH! → Send to Express
Check Route 2: ^/odata/(.*)$        ──► (never reached)
Check Route 3: ^/animaladoption/..  ──► (never reached)
Check Route 4: ^/sap/(.*)$          ──► (never reached)
Check Route 5: ^(.*)$               ──► (never reached)
```

```
Request: /style.css

Check Route 1: ^/api/(.*)$          ──► No match
Check Route 2: ^/odata/(.*)$        ──► No match
Check Route 3: ^/animaladoption/..  ──► No match
Check Route 4: ^/sap/(.*)$          ──► No match
Check Route 5: ^(.*)$               ──► MATCH! → Serve from webapp/
```

> **ALWAYS put the catch-all route LAST!** If you put it first, it matches everything and no other route ever fires.

## Regex Cheatsheet for xs-app.json

| Pattern | Meaning | Example |
|---------|---------|---------|
| `^` | Start of string | `^/api` = must start with /api |
| `$` | End of string | `api$` = must end with api |
| `(.*)` | Capture everything | `/api/(.*)` captures everything after /api/ |
| `$1` | Reference captured group | Use in `target` to insert the captured part |

---

# Chapter 6: Understanding Destinations

## What is a Destination?

A **destination** is a named pointer to a backend URL. Think of it as a contact in your phone:

```
Phone Contact:        "Mom"     →  +91-9876543210
Destination:   "microservice_api" →  http://localhost:4040
```

In `xs-app.json` routes, you write `"destination": "microservice_api"` — the approuter looks up the actual URL from the destination config.

## Where Are Destinations Defined?

### Locally: `default-env.json`

```json
{
  "destinations": [
    {
      "name": "microservice_api",
      "url": "http://localhost:4040",
      "forwardAuthToken": false
    },
    {
      "name": "cap_api",
      "url": "http://localhost:4004",
      "forwardAuthToken": false
    }
  ]
}
```

| Field | Meaning |
|-------|---------|
| `name` | Must match the `destination` value in xs-app.json routes |
| `url` | The actual backend URL |
| `forwardAuthToken` | `true` = send JWT token to backend, `false` = don't |

### On Cloud: Environment Variable in `manifest.yaml`

```yaml
env:
  destinations: >
    [
      {
        "name": "microservice_api",
        "url": "https://deep-microservice.cfapps.us10-001.hana.ondemand.com",
        "forwardAuthToken": true
      }
    ]
```

### On Cloud (MTA): Auto-wired via `provides` / `requires`

```yaml
# Backend provides its URL
- name: deep-microservice
  provides:
    - name: microservice-api
      properties:
        srv-url: ${default-url}    # CF auto-fills the actual URL

# Approuter requires that URL
- name: deep-approuter
  requires:
    - name: microservice-api
      group: destinations
      properties:
        name: microservice_api     # This becomes the destination name
        url: ~{srv-url}            # This gets the auto-filled URL
```

## How the Destination Lookup Works

```
1. Browser hits:   http://localhost:5000/api/data.json

2. xs-app.json:    source "^/api/(.*)$" → destination "microservice_api"

3. Approuter asks: "What's the URL for 'microservice_api'?"

4. default-env.json: "microservice_api" → "http://localhost:4040"

5. Approuter sends: http://localhost:4040/data.json

6. Express responds: { "employees": [...] }

7. Approuter returns response to browser
```

## The Difference Between Local and Cloud Destinations

| | Local (`default-env.json`) | Cloud (`manifest.yaml` or MTA) |
|---|---|---|
| URLs | `http://localhost:PORT` | `https://app-name.cfapps.region.hana.ondemand.com` |
| `forwardAuthToken` | `false` (no JWT locally) | `true` (forward the XSUAA JWT) |
| Created by | You, manually | You (manifest) or auto-wired (MTA) |

---

# Chapter 7: Understanding xs-security.json

## What is XSUAA?

**XSUAA** = Extended Services for User Account and Authentication

Think of it as BTP's **login system**. When a user accesses your app:

```
1. User hits approuter
2. Approuter says: "You need to login first!"
3. User is redirected to BTP login page
4. User enters email + password
5. BTP gives back a JWT token (like an ID card)
6. Approuter stores the token and lets the user in
7. For every backend request, approuter sends the JWT token
8. Backend checks: "Does this token have the right role?"
```

## xs-security.json Explained

```json
{
  "xsappname": "deep-approuter-app",
  "tenant-mode": "dedicated",
  "scopes": [
    {
      "name": "$XSAPPNAME.Adopter",
      "description": "Adopter role"
    },
    {
      "name": "$XSAPPNAME.Admin",
      "description": "Admin role"
    }
  ],
  "role-templates": [
    {
      "name": "Adopter",
      "scope-references": ["$XSAPPNAME.Adopter"]
    },
    {
      "name": "Admin",
      "scope-references": ["$XSAPPNAME.Admin"]
    }
  ],
  "role-collections": [
    {
      "name": "AnimalAdoption_Adopter",
      "role-template-references": ["$XSAPPNAME.Adopter"]
    },
    {
      "name": "AnimalAdoption_Admin",
      "role-template-references": ["$XSAPPNAME.Admin"]
    }
  ]
}
```

## The Hierarchy (Bottom-Up)

```
                    ┌──────────────────────┐
                    │   Role Collection    │  ← Assigned to USERS
                    │ "AnimalAdoption_     │     in BTP Cockpit
                    │       Admin"         │
                    └─────────┬────────────┘
                              │ contains
                    ┌─────────▼────────────┐
                    │   Role Template      │  ← Groups of scopes
                    │     "Admin"          │
                    └─────────┬────────────┘
                              │ references
                    ┌─────────▼────────────┐
                    │      Scope           │  ← Individual permissions
                    │ "$XSAPPNAME.Admin"   │
                    └──────────────────────┘
```

### Real-World Analogy:

| Concept | Real World | Our App |
|---------|-----------|---------|
| **Scope** | Permission to enter a room | `$XSAPPNAME.Admin` = permission to manage |
| **Role Template** | Job title (defines which rooms you can enter) | `Admin` = can manage animals + applications |
| **Role Collection** | ID badge (given to a person) | `AnimalAdoption_Admin` = assigned to user "deep" |

### How `$XSAPPNAME` Works

`$XSAPPNAME` is automatically replaced with your app name at runtime:

```
In xs-security.json:   $XSAPPNAME.Admin
After deployment:      deep-approuter-app.Admin
```

This prevents scope name collisions between different apps on the same BTP subaccount.

## How It Connects to CAP

In your CAP service (`service.cds`), you write:

```cds
entity Animals @(restrict:[
    { grant: ['READ'], to: ['Adopter'] },
    { grant: ['*'], to: ['Admin'] }
])
```

The values `'Adopter'` and `'Admin'` must match the **scope names** in `xs-security.json` (without the `$XSAPPNAME.` prefix — CDS adds it automatically).

## Local vs Cloud Authentication

| | Local | Cloud |
|---|---|---|
| Auth method | `"authenticationMethod": "none"` | `"authenticationMethod": "route"` |
| Auth type per route | `"authenticationType": "none"` | `"authenticationType": "xsuaa"` |
| CAP auth | `kind: "mocked"` with hardcoded users | `kind: "xsuaa"` with real JWT tokens |
| Login | No login needed | BTP login page |

### Local Mocked Users (in CAP package.json)

```json
"[development]": {
  "auth": {
    "kind": "mocked",
    "users": {
      "deep": {
        "roles": ["Admin"],
        "password": "deep"
      }
    }
  }
}
```

When accessing CAP locally, browser prompts for Basic Auth: username `deep`, password `deep`.

---

# Chapter 8: Running Locally

## Quick Start (3 Terminals)

```
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│  Terminal 1       │  │  Terminal 2       │  │  Terminal 3       │
│                   │  │                   │  │                   │
│  cd node_micro_   │  │  cd CAP_annota.. │  │  cd approuter     │
│     service       │  │                   │  │                   │
│  npm start        │  │  cds watch        │  │  npm start        │
│                   │  │                   │  │                   │
│  ► port 4040      │  │  ► port 4004      │  │  ► port 5000      │
└──────────────────┘  └──────────────────┘  └──────────────────┘
```

## Checklist Before Running

| # | Check | File |
|---|-------|------|
| 1 | `xs-app.json` has `authenticationMethod: "none"` | `approuter/xs-app.json` |
| 2 | All routes have `authenticationType: "none"` | `approuter/xs-app.json` |
| 3 | `default-env.json` exists with correct ports | `approuter/default-env.json` |
| 4 | `npm install` done in all 3 folders | All `package.json` |

## Testing URLs

| What to Test | URL | Expected Result |
|---|---|---|
| Approuter landing page | `http://localhost:5000` | index.html with links |
| Express via approuter | `http://localhost:5000/api/` | "Hello World" |
| Express data via approuter | `http://localhost:5000/api/data.json` | JSON data |
| CAP OData direct | `http://localhost:5000/odata/v4/adoptor/` | OData service document |
| CAP Animals entity | `http://localhost:5000/odata/v4/adoptor/Animals` | List of animals |
| CAP via prefix | `http://localhost:5000/animaladoption/odata/v4/adoptor/` | Same OData service |
| Express directly (bypass) | `http://localhost:4040` | "Hello World" |
| CAP directly (bypass) | `http://localhost:4004` | CDS welcome page |

---

# Chapter 9: Cloud Deployment with manifest.yaml

## What is manifest.yaml?

A **deployment descriptor** for Cloud Foundry. It tells CF: "Here are my apps, how much memory they need, and what services they use."

## Our manifest.yaml Explained

```yaml
---
applications:
  # ---- Backend 1: Express Microservice ----
  - name: deep-microservice              # App name on CF
    path: node_micro_service             # Which local folder to upload
    memory: 256M                         # RAM limit
    instances: 1                         # How many copies to run
    buildpacks:
      - nodejs_buildpack                 # CF knows it's a Node.js app
    command: npm start                   # How to start the app
    random-route: true                   # CF generates a random URL

  # ---- Backend 2: CAP Application ----
  - name: deep-cap-srv
    path: CAP_annotation_based_Application_local
    memory: 256M
    instances: 1
    buildpacks:
      - nodejs_buildpack
    command: npx cds-serve               # Start CAP server
    random-route: true
    services:
      - deep-xsuaa-service               # Bind XSUAA for authentication

  # ---- Entry Point: Approuter ----
  - name: deep-approuter
    path: approuter
    memory: 256M
    instances: 1
    buildpacks:
      - nodejs_buildpack
    env:
      destinations: >                    # JSON string as env variable
        [
          {
            "name": "microservice_api",
            "url": "https://deep-microservice.cfapps.us10-001.hana.ondemand.com",
            "forwardAuthToken": true
          },
          {
            "name": "cap_api",
            "url": "https://deep-cap-srv.cfapps.us10-001.hana.ondemand.com",
            "forwardAuthToken": true
          }
        ]
    services:
      - deep-xsuaa-service               # Bind XSUAA for login
    random-route: true
```

## What is `services:` ?

```yaml
services:
  - deep-xsuaa-service
```

This **binds** a service instance to the app. It's like plugging in a cable:

```
┌────────────┐         ┌──────────────────┐
│ XSUAA      │ ◄─bind──│  deep-approuter   │
│ Service    │         │  deep-cap-srv     │
│ Instance   │         └──────────────────┘
└────────────┘
  Created from
  xs-security.json
```

## Step-by-Step Cloud Deployment

### Step 1: Login to CF

```bash
cf login -a https://api.cf.us10-001.hana.ondemand.com
# Enter email and password
```

### Step 2: Create XSUAA Service (one-time)

```bash
cf create-service xsuaa application deep-xsuaa-service -c xs-security.json
```

> **Important:** Run this from the ROOT folder where `xs-security.json` is located!

### Step 3: Switch xs-app.json to Production Mode

Change in `approuter/xs-app.json`:
```json
"authenticationMethod": "route"
// And every route:
"authenticationType": "xsuaa"
```

### Step 4: Deploy

```bash
cf push
```

This deploys all 3 apps at once.

### Step 5: Get the Actual URLs

```bash
cf apps
```

Output:
```
name               requested state   routes
deep-microservice  started           deep-microservice-happy-fox.cfapps.us10-001...
deep-cap-srv       started           deep-cap-srv-sleepy-cat.cfapps.us10-001...
deep-approuter     started           deep-approuter-brave-dog.cfapps.us10-001...
```

### Step 6: Update Destinations with Real URLs

Replace `<YOUR_CF_LANDSCAPE>` placeholders in `manifest.yaml` with the actual URLs from Step 5, then:

```bash
cf push deep-approuter   # Redeploy only the approuter
```

### Step 7: Assign Roles to Users

1. Open **BTP Cockpit** → Security → Role Collections
2. Find `AnimalAdoption_Admin`
3. Add your email address
4. Save

### Limitations of manifest.yaml

| Problem | Why |
|---------|-----|
| **Hardcoded URLs** | You must manually copy-paste backend URLs into destinations |
| **Manual service creation** | Must run `cf create-service` before `cf push` |
| **No dependency ordering** | CF deploys apps in parallel — approuter might start before backends |
| **Two-step deploy** | Push → get URLs → update manifest → push again |

---

# Chapter 10: Cloud Deployment with mta.yaml

## Why MTA is Better

**MTA** = Multi-Target Application. It solves ALL the manifest.yaml problems:

| manifest.yaml Problem | MTA Solution |
|----------------------|--------------|
| Hardcoded URLs | **Auto-resolved** via `provides`/`requires` |
| Manual service creation | **Automatic** via `resources` section |
| No dependency ordering | **Built-in** dependency management |
| Two-step deploy | **Single command** deployment |

## The mta.yaml File

```yaml
_schema-version: "3.1"
ID: deep-approuter-app
version: 1.0.0
description: Deep Approuter Multi-Target Application

parameters:
  enable-parallel-deployments: true

modules:
  # ---- Backend 1: Express Microservice ----
  - name: deep-microservice
    type: nodejs
    path: node_micro_service
    parameters:
      memory: 256M
      instances: 1
      buildpack: nodejs_buildpack
    provides:                           # ← "I provide a URL"
      - name: microservice-api
        properties:
          srv-url: ${default-url}       # ← CF fills in real URL

  # ---- Backend 2: CAP Server ----
  - name: deep-cap-srv
    type: nodejs
    path: CAP_annotation_based_Application_local
    parameters:
      memory: 256M
      instances: 1
      buildpack: nodejs_buildpack
    requires:
      - name: deep-xsuaa-service       # ← Needs XSUAA
    provides:                           # ← "I provide a URL"
      - name: cap-api
        properties:
          srv-url: ${default-url}

  # ---- Entry Point: Approuter ----
  - name: deep-approuter
    type: approuter.nodejs
    path: approuter
    parameters:
      memory: 256M
      instances: 1
    requires:
      - name: deep-xsuaa-service       # ← Needs XSUAA for login
      - name: microservice-api          # ← Needs microservice URL
        group: destinations
        properties:
          name: microservice_api        # ← Destination name (matches xs-app.json)
          url: ~{srv-url}              # ← Auto-filled URL from provides
          forwardAuthToken: true
      - name: cap-api                   # ← Needs CAP URL
        group: destinations
        properties:
          name: cap_api
          url: ~{srv-url}
          forwardAuthToken: true

resources:
  # ---- XSUAA Service (auto-created) ----
  - name: deep-xsuaa-service
    type: org.cloudfoundry.managed-service
    parameters:
      service: xsuaa
      service-plan: application
      path: xs-security.json
```

## How `provides` / `requires` Works

```
Step 1: MTA deploys deep-microservice
        CF assigns URL: deep-microservice-abc123.cfapps.us10-001.hana.ondemand.com
        
Step 2: provides captures the URL
        microservice-api.srv-url = "https://deep-microservice-abc123.cfapps..."
        
Step 3: MTA deploys deep-approuter  
        requires looks up microservice-api
        
Step 4: ~{srv-url} is replaced with the actual URL
        destinations env var is auto-generated:
        [{"name":"microservice_api","url":"https://deep-microservice-abc123...","forwardAuthToken":true}]
```

```
┌──────────────────────┐          ┌─────────────────────────┐
│  deep-microservice   │          │    deep-approuter       │
│                      │          │                         │
│  provides:           │  ─────►  │  requires:              │
│    microservice-api  │  auto    │    microservice-api     │
│    srv-url =         │  wired   │    group: destinations  │
│    ${default-url}    │          │    url: ~{srv-url}      │
└──────────────────────┘          └─────────────────────────┘
```

## MTA Deployment Steps

```bash
# 1. Install MTA Build Tool (one-time)
npm install -g mbt

# 2. Install CF MTA plugin (one-time)
cf install-plugin multiapps

# 3. Login
cf login -a https://api.cf.us10-001.hana.ondemand.com

# 4. Build the .mtar archive
mbt build

# 5. Deploy everything in one command!
cf deploy mta_archives/deep-approuter-app_1.0.0.mtar
```

**That's it!** No manual service creation, no URL copy-pasting, no two-step deploy.

## manifest.yaml vs mta.yaml — Summary

| Feature | manifest.yaml | mta.yaml |
|---------|--------------|----------|
| Complexity | Simple | More config but more powerful |
| Service creation | Manual (`cf create-service`) | **Automatic** |
| Destination URLs | **Hardcoded** | **Auto-resolved** |
| Deploy command | `cf push` | `mbt build` + `cf deploy` |
| Dependency order | None | **Automatic** |
| Recommended for | Learning / simple apps | **Production apps** |

---

# Chapter 11: Common Errors & Fixes

## Error 1: `404 ENOENT /sap/bc/lrep/flex/...`

```
GET request to /sap/bc/lrep/flex/data/animaladoption completed with status 404
ENOENT: no such file or directory
```

**Why:** Fiori Elements UI calls `/sap/bc/lrep/*` internally. The catch-all route sends it to `webapp/` folder which doesn't have these files.

**Fix:** Add a `/sap/*` route BEFORE the catch-all:
```json
{
  "source": "^/sap/(.*)$",
  "target": "/sap/$1",
  "destination": "cap_api",
  "authenticationType": "none"
}
```

## Error 2: `504 Timeout on $batch`

```
POST request to /odata/v4/adoptor/$batch completed with status 504
Request failed with a timeout
```

**Why:** CAP's mocked auth expects Basic Auth credentials (username/password), but the approuter doesn't send any.

**Fix:** Add Basic Auth header in `default-env.json`:
```json
{
  "name": "cap_api",
  "url": "http://localhost:4004",
  "forwardAuthToken": false,
  "httpHeaders": {
    "Authorization": "Basic ZGVlcDpkZWVw"
  }
}
```
> `ZGVlcDpkZWVw` = base64 of `deep:deep` (username:password)

## Error 3: `Cannot find module @cap-js/sqlite`

```
Error: Failed loading service implementation from @cap-js/sqlite
Cannot find module '/home/vcap/app/@cap-js/sqlite'
```

**Why:** `@cap-js/sqlite` is a `devDependency`. CF sets `NPM_CONFIG_PRODUCTION=true` so devDependencies are NOT installed.

**Fix:** Move `@cap-js/sqlite` from `devDependencies` to `dependencies`:
```json
"dependencies": {
  "@cap-js/sqlite": "^2",
  "@sap/cds": "^9"
}
```

And scope the db config per profile:
```json
"[development]": {
  "db": { "kind": "sqlite" }
},
"[production]": {
  "db": { "kind": "sqlite" }
}
```

## Error 4: `Routes quota exceeded`

```
Routes quota exceeded for organization 'trial'
```

**Why:** BTP trial has a limited number of routes (usually 2). Deploying 3 apps with `random-route: true` exceeds it.

**Fix:**
```bash
# Delete old unused routes
cf delete-orphaned-routes -f

# Or set no-route for backends (only approuter needs a public route)
# In manifest.yaml:
- name: deep-microservice
  no-route: true         # No public URL needed
```

## Error 5: `Invalid xsappname: Must not be null`

```
Error parsing xs-security.json data: Invalid xsappname: Must not be <null>
```

**Why:** You ran `cf create-service` from the WRONG folder — it picked up the CAP app's `xs-security.json` which has no `xsappname`.

**Fix:** Run from the ROOT folder:
```bash
cd deepappRouter          # root, not CAP_annotation...
cf create-service xsuaa application deep-xsuaa-service -c xs-security.json
```

---

# Chapter 12: Quick Reference Cheatsheet

## File Purpose Map

| File | Location | Purpose |
|------|----------|---------|
| `package.json` | `approuter/` | Only dep: `@sap/approuter` |
| `xs-app.json` | `approuter/` | URL → destination routing rules |
| `default-env.json` | `approuter/` | Local destination URLs |
| `webapp/` | `approuter/` | Static HTML/CSS/JS files |
| `xs-security.json` | Root | Scopes, roles, role-collections |
| `manifest.yaml` | Root | CF deployment (simple) |
| `mta.yaml` | Root | CF deployment (recommended) |

## xs-app.json Routing Patterns

| Pattern | Target | Effect |
|---------|--------|--------|
| `"source": "^/api/(.*)$"` + `"target": "/$1"` | **Strip prefix** | `/api/data` → `/data` |
| `"source": "^/odata/(.*)$"` + `"target": "/odata/$1"` | **Keep prefix** | `/odata/v4/` → `/odata/v4/` |
| `"source": "^(.*)$"` + `"localDir": "webapp"` | **Serve static** | `/index.html` → `webapp/index.html` |

## Local vs Cloud Config Switch

| Setting | Local | Cloud |
|---------|-------|-------|
| `authenticationMethod` | `"none"` | `"route"` |
| `authenticationType` | `"none"` | `"xsuaa"` |
| `forwardAuthToken` | `false` | `true` |
| CAP auth kind | `"mocked"` | `"xsuaa"` |

## Essential Commands

```bash
# ── Local ──
cd approuter && npm install      # Install approuter deps
cd approuter && npm start        # Start approuter (port 5000)
cd node_micro_service && npm start  # Start Express (port 4040)
cd CAP_annotation... && cds watch   # Start CAP (port 4004)

# ── Cloud (manifest.yaml) ──
cf login                         # Login to CF
cf create-service xsuaa application deep-xsuaa-service -c xs-security.json
cf push                          # Deploy all apps
cf apps                          # List apps and URLs
cf logs <app-name> --recent      # Check logs

# ── Cloud (mta.yaml) ──
npm install -g mbt               # Install MTA build tool
cf install-plugin multiapps      # Install CF MTA plugin
mbt build                        # Build .mtar archive
cf deploy mta_archives/*.mtar    # Deploy everything
```

---

# Chapter 13: FAQ — Multi-App xs-security.json (Roles & Scopes)

## The Big Question: 5 Apps, 1 Approuter — What Goes in xs-security.json?

Imagine you have 5 backend apps behind one approuter:

```
                        ┌─────────────────┐
                        │   APPROUTER     │
                        │  (deep-approuter)│
                        └──┬──┬──┬──┬──┬──┘
                           │  │  │  │  │
              ┌────────────┘  │  │  │  └────────────┐
              ▼               ▼  ▼  ▼               ▼
         App 1: HR       App 2  App 3  App 4    App 5: Finance
         Roles:          Animal  Inventory  CRM   Roles:
         - Employee      Adopt.  Roles:   Roles:  - Accountant
         - HRManager     Roles:  -Warehouse -Sales - FinanceAdmin
                         -Adopter -InvAdmin -SalesMgr
                         -Admin
```

Each app has its own roles. Some roles are **common** (like `Admin`), some are **unique** (like `Warehouse`).

## Answer: There Are 2 Approaches

---

### Approach 1: ONE Shared XSUAA Instance (Recommended for Approuter)

**Rule: ALL scopes from ALL 5 apps go into ONE xs-security.json.**

The approuter and all backends bind to the **same** XSUAA instance. This is the standard approach.

```
ONE xs-security.json ──► ONE XSUAA Instance ──► bound to ALL 6 apps
                                                  (1 approuter + 5 backends)
```

#### The xs-security.json (Merged)

```json
{
  "xsappname": "deep-multi-app",
  "tenant-mode": "dedicated",
  "scopes": [
    // ─── Common Scopes (shared by multiple apps) ───
    {
      "name": "$XSAPPNAME.Admin",
      "description": "Global admin access across all apps"
    },

    // ─── App 1: HR ───
    {
      "name": "$XSAPPNAME.Employee",
      "description": "HR - View own records"
    },
    {
      "name": "$XSAPPNAME.HRManager",
      "description": "HR - Manage all employee records"
    },

    // ─── App 2: Animal Adoption ───
    {
      "name": "$XSAPPNAME.Adopter",
      "description": "Animal Adoption - View and adopt animals"
    },

    // ─── App 3: Inventory ───
    {
      "name": "$XSAPPNAME.Warehouse",
      "description": "Inventory - View stock"
    },
    {
      "name": "$XSAPPNAME.InvAdmin",
      "description": "Inventory - Full management"
    },

    // ─── App 4: CRM ───
    {
      "name": "$XSAPPNAME.Sales",
      "description": "CRM - View customers and deals"
    },
    {
      "name": "$XSAPPNAME.SalesMgr",
      "description": "CRM - Manage sales pipeline"
    },

    // ─── App 5: Finance ───
    {
      "name": "$XSAPPNAME.Accountant",
      "description": "Finance - View financial records"
    },
    {
      "name": "$XSAPPNAME.FinanceAdmin",
      "description": "Finance - Manage all financial data"
    }
  ],

  "role-templates": [
    // ─── Common ───
    { "name": "Admin",        "scope-references": ["$XSAPPNAME.Admin"] },

    // ─── App 1: HR ───
    { "name": "Employee",     "scope-references": ["$XSAPPNAME.Employee"] },
    { "name": "HRManager",    "scope-references": ["$XSAPPNAME.HRManager"] },

    // ─── App 2: Animal Adoption ───
    { "name": "Adopter",      "scope-references": ["$XSAPPNAME.Adopter"] },

    // ─── App 3: Inventory ───
    { "name": "Warehouse",    "scope-references": ["$XSAPPNAME.Warehouse"] },
    { "name": "InvAdmin",     "scope-references": ["$XSAPPNAME.InvAdmin"] },

    // ─── App 4: CRM ───
    { "name": "Sales",        "scope-references": ["$XSAPPNAME.Sales"] },
    { "name": "SalesMgr",     "scope-references": ["$XSAPPNAME.SalesMgr"] },

    // ─── App 5: Finance ───
    { "name": "Accountant",   "scope-references": ["$XSAPPNAME.Accountant"] },
    { "name": "FinanceAdmin", "scope-references": ["$XSAPPNAME.FinanceAdmin"] }
  ],

  "role-collections": [
    // ─── Super Admin (all apps) ───
    {
      "name": "SuperAdmin",
      "description": "Full access to ALL applications",
      "role-template-references": [
        "$XSAPPNAME.Admin"
      ]
    },

    // ─── Per-App Role Collections ───
    {
      "name": "HR_Employee",
      "description": "HR employees - view own records",
      "role-template-references": ["$XSAPPNAME.Employee"]
    },
    {
      "name": "HR_Manager",
      "description": "HR managers - manage all employee records",
      "role-template-references": ["$XSAPPNAME.Employee", "$XSAPPNAME.HRManager"]
    },
    {
      "name": "AnimalAdoption_Adopter",
      "description": "Can view and adopt animals",
      "role-template-references": ["$XSAPPNAME.Adopter"]
    },
    {
      "name": "AnimalAdoption_Admin",
      "description": "Full animal adoption management",
      "role-template-references": ["$XSAPPNAME.Admin"]
    },
    {
      "name": "Inventory_Staff",
      "description": "Warehouse staff - view stock",
      "role-template-references": ["$XSAPPNAME.Warehouse"]
    },
    {
      "name": "Inventory_Admin",
      "description": "Inventory managers",
      "role-template-references": ["$XSAPPNAME.Warehouse", "$XSAPPNAME.InvAdmin"]
    },
    {
      "name": "CRM_SalesPerson",
      "description": "View customers and deals",
      "role-template-references": ["$XSAPPNAME.Sales"]
    },
    {
      "name": "CRM_SalesManager",
      "description": "Manage entire sales pipeline",
      "role-template-references": ["$XSAPPNAME.Sales", "$XSAPPNAME.SalesMgr"]
    },
    {
      "name": "Finance_Accountant",
      "description": "View financial records",
      "role-template-references": ["$XSAPPNAME.Accountant"]
    },
    {
      "name": "Finance_Admin",
      "description": "Full finance management",
      "role-template-references": ["$XSAPPNAME.Accountant", "$XSAPPNAME.FinanceAdmin"]
    },

    // ─── Cross-App Combined Collection ───
    {
      "name": "OperationsManager",
      "description": "Manages Inventory + CRM + Finance",
      "role-template-references": [
        "$XSAPPNAME.InvAdmin",
        "$XSAPPNAME.SalesMgr",
        "$XSAPPNAME.FinanceAdmin"
      ]
    }
  ]
}
```

#### Key Insights

| Concept | Example | Meaning |
|---------|---------|---------|
| **Common scope** | `Admin` | Used by multiple apps (Animal Adoption + others check `to: 'Admin'`) |
| **App-specific scope** | `Warehouse` | Only Inventory app uses this |
| **Combined role-collection** | `OperationsManager` | Bundles roles from 3 different apps into one assignment |
| **Layered role-collection** | `HR_Manager` | Gets both `Employee` + `HRManager` scopes (managers can do everything employees can + more) |

---

### Approach 2: Separate XSUAA Per App (Advanced — Foreign Scope References)

In large enterprise scenarios, each app has its **own XSUAA service instance** with its own `xs-security.json`. Apps that need to call each other use **foreign scope references**.

```
xs-security-hr.json    ──► hr-xsuaa         ──► HR App
xs-security-adopt.json ──► adopt-xsuaa      ──► Animal Adoption App
xs-security-inv.json   ──► inv-xsuaa        ──► Inventory App

Approuter binds to ONE of them (usually its own)
Other apps grant scopes to the approuter's XSUAA using "granted-apps"
```

#### How Foreign Scope References Work

**App 2 (Animal Adoption) xs-security.json:**
```json
{
  "xsappname": "animal-adoption-app",
  "scopes": [
    {
      "name": "$XSAPPNAME.Adopter",
      "granted-apps": ["deep-approuter-app"]
    }
  ]
}
```

**Approuter xs-security.json:**
```json
{
  "xsappname": "deep-approuter-app",
  "scopes": [
    {
      "name": "$XSAPPNAME.Admin",
      "description": "Local admin scope"
    }
  ],
  "foreign-scope-references": {
    "name": "$XSAPPNAME(application, animal-adoption-app).Adopter"
  }
}
```

The `"granted-apps"` field says: "I allow the `deep-approuter-app` XSUAA to access my `Adopter` scope."

#### When to Use Which Approach?

| Criteria | Approach 1 (Single XSUAA) | Approach 2 (Foreign Scopes) |
|----------|--------------------------|----------------------------|
| **Complexity** | Simple | Complex |
| **Best for** | Small-medium projects, learning | Enterprise with separate teams |
| **# of apps** | Up to ~10 | 10+ with separate lifecycles |
| **Deployment** | All apps share 1 XSUAA | Each app has its own XSUAA |
| **Role management** | All roles in one place | Distributed across teams |
| **Our project** | **✓ Use this** | Overkill for our scenario |

---

## FAQ: Common Questions

### Q1: If 2 apps both need an "Admin" role, do I create 2 scopes?

**No!** Create ONE scope and both apps use the same one:

```json
// ONE scope in xs-security.json
{ "name": "$XSAPPNAME.Admin" }
```

```cds
// App 2 (Animal Adoption) service.cds
entity Animals @(restrict: [{ grant: '*', to: 'Admin' }])

// App 3 (Inventory) service.cds
entity Products @(restrict: [{ grant: '*', to: 'Admin' }])
```

Both check for the same `Admin` scope from the same JWT token. A user with `Admin` role collection gets access to **both** apps.

### Q2: But what if I want separate admins per app?

Then use **prefixed scope names**:

```json
{
  "scopes": [
    { "name": "$XSAPPNAME.HR_Admin" },
    { "name": "$XSAPPNAME.Adopt_Admin" },
    { "name": "$XSAPPNAME.Inv_Admin" }
  ]
}
```

```cds
// App 1 (HR) service.cds
@requires: 'HR_Admin'

// App 2 (Animal Adoption) service.cds
@requires: 'Adopt_Admin'

// App 3 (Inventory) service.cds
@requires: 'Inv_Admin'
```

Now you can assign `HR_Admin` to HR managers only, and `Inv_Admin` to warehouse managers only.

### Q3: How do role-collections help with multiple apps?

Role collections are what you **assign to actual users** in BTP Cockpit. They can bundle roles from **multiple apps**:

```
User: "Ramesh" gets role collection "OperationsManager"

OperationsManager includes:
  ├── InvAdmin    → Ramesh can manage inventory
  ├── SalesMgr    → Ramesh can manage CRM
  └── FinanceAdmin → Ramesh can manage finance

But NOT:
  ├── HRManager   → Ramesh CANNOT access HR
  └── Adopter     → Ramesh CANNOT access Animal Adoption
```

This is the **power of role-collections** — one assignment, multiple app access.

### Q4: Does the CAP app's own xs-security.json matter?

**On Cloud: NO.** Only the xs-security.json used to create the XSUAA service instance matters. The CAP app's own file is **ignored at runtime**.

```
What matters:                          What is ignored:
Root xs-security.json                  CAP_annotation.../xs-security.json
  └── used by: cf create-service       └── reference only, not deployed
```

**However**, during `cds build` or when using `cds compile --to xsuaa`, CDS can **generate** an xs-security.json from your `@restrict` annotations. This generated file is useful as a **starting point** — but you should merge it into the root file.

### Q5: What if I add a new app later?

Just update the root `xs-security.json`:

1. Add the new scopes
2. Add new role-templates
3. Add new role-collections
4. **Update** the XSUAA service:

```bash
cf update-service deep-xsuaa-service -c xs-security.json
```

> **Important:** Use `cf update-service` (not `create-service`). This updates the existing instance without deleting it.

### Q6: Can I auto-generate xs-security.json from CDS models?

**Yes!** CAP can extract the roles from your `service.cds` annotations:

```bash
cd CAP_annotation_based_Application_local
cds compile srv/ --to xsuaa > generated-xs-security.json
```

This outputs something like:
```json
{
  "scopes": [
    { "name": "$XSAPPNAME.Adopter" },
    { "name": "$XSAPPNAME.Admin" }
  ],
  "role-templates": [
    { "name": "Adopter", "scope-references": ["$XSAPPNAME.Adopter"] },
    { "name": "Admin", "scope-references": ["$XSAPPNAME.Admin"] }
  ]
}
```

Then you **merge** this into your root xs-security.json (add `xsappname`, `tenant-mode`, `role-collections`, and scopes from other apps).

### Q7: Naming Convention Best Practices

| Pattern | Example | When to Use |
|---------|---------|-------------|
| `AppName_Role` | `HR_Admin`, `CRM_Sales` | When same role name in multiple apps |
| `Role` (simple) | `Admin`, `Adopter` | When role is unique or shared globally |
| `AppPrefix.Action` | `Inv.Read`, `Inv.Write` | Very granular (not recommended by SAP) |

**SAP Best Practice:** Use **conceptual business roles** (`Vendor`, `Customer`, `Accountant`), not technical operations (`Read`, `Write`, `Delete`).

### Q8: Maximum Limits

| Resource | Limit (per XSUAA instance) |
|----------|---------------------------|
| Scopes | 200 |
| Role Templates | 200 |
| Role Collections | 1000 |
| Attributes | 30 |

For most projects, you'll never hit these limits. If you do, consider Approach 2 (separate XSUAA instances).

---

## Visual Summary: The Complete Picture

```
┌──────────────────────────────────────────────────────────┐
│                    BTP Cockpit                           │
│                                                          │
│  Role Collections (assigned to USERS):                   │
│  ┌─────────────────────┐  ┌──────────────────────────┐  │
│  │ AnimalAdoption_Admin │  │ OperationsManager        │  │
│  │  └─ Admin role-tmpl  │  │  └─ InvAdmin role-tmpl   │  │
│  │                      │  │  └─ SalesMgr role-tmpl   │  │
│  │  Assigned to: "deep" │  │  └─ FinanceAdmin role    │  │
│  └─────────────────────┘  │                           │  │
│                            │  Assigned to: "ramesh"    │  │
│                            └──────────────────────────┘  │
└──────────────────────────┬───────────────────────────────┘
                           │
                    JWT Token contains:
                    { scopes: ["Admin"] }
                    or
                    { scopes: ["InvAdmin","SalesMgr","FinanceAdmin"] }
                           │
                           ▼
                ┌────────────────────┐
                │    APPROUTER       │
                │  Validates JWT     │
                │  Forwards token    │
                └──┬─────────┬───┬──┘
                   │         │   │
                   ▼         ▼   ▼
              ┌────────┐ ┌────────┐ ┌────────┐
              │ App 1  │ │ App 2  │ │ App 3  │
              │ Checks:│ │ Checks:│ │ Checks:│
              │ Admin? │ │ Adopter│ │InvAdmin│
              └────────┘ │ Admin? │ └────────┘
                         └────────┘
```

---

## You Made It!

You now understand:
- **What** the approuter is (security guard + receptionist)
- **Why** we need it (single entry point, central auth, no CORS)
- **How** `xs-app.json` routes work (regex matching, top-to-bottom)
- **How** destinations connect routes to backends
- **How** `xs-security.json` defines roles and permissions
- **How** to handle multiple apps with common and different roles
- **How** to run locally (3 terminals, `authenticationMethod: "none"`)
- **How** to deploy with `manifest.yaml` (simple but manual)
- **How** to deploy with `mta.yaml` (auto-wired, recommended)
- **How** to debug common errors

Happy coding! 🚀
