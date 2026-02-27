# 🔐 Phase 3: Authorization Test Report

**Target System:** `http://localhost:8004`  
**Tester:** Dan Luchin

---

## 1️⃣ Executive Summary
This report evaluates the access control mechanisms of the Phase 3 implementation against the provided official specifications. Testing was conducted via manual browser inspection, automated OWASP ZAP scanning, and directory brute-forcing (Gobuster/Wfuzz).

---

## 2️⃣ Role-Based Access Matrix

### Endpoint discovery using Gobuster

I have downloaded a common.txt file which has common potential names for testing

**gobuster test against the `http://localhost:8004` path.**

```
PS C:\Users\luchi\Downloads\gobuster_Windows_x86_64> .\gobuster.exe dir -u http://localhost:8004 -w common.txt
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://localhost:8004
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
login                (Status: 200) [Size: 2131]
logout               (Status: 302) [Size: 0] [--> /]
register             (Status: 200) [Size: 3098]
reservation          (Status: 303) [Size: 0] [--> /status.html?status=failed&message=%3Cstrong%3EUnauthorized%3C%2Fstrong%3E]
resources            (Status: 200) [Size: 2473]
Progress: 4751 / 4751 (100.00%)
===============================================================
Finished
===============================================================
```

**gobuster test against the `http://localhost:8004/api` path.**

```
PS C:\Users\luchi\Downloads\gobuster_Windows_x86_64> .\gobuster.exe dir -u http://localhost:8004/api -w common.txt
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://localhost:8004/api
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
reservations         (Status: 200) [Size: 304]
resources            (Status: 200) [Size: 160]
session              (Status: 401) [Size: 24]
users                (Status: 200) [Size: 303]
Progress: 4751 / 4751 (100.00%)
===============================================================
Finished
===============================================================
```

### 🧑‍🦲 Guest (Unauthenticated)
*Goal: Verify public access only.*

| Action | Endpoint | Observation | Spec Match |
| :--- | :--- | :--- | :--- |
| **✅ Can do** | | | |
| View public resource list | `/` | Successfully lists resources. | Yes |
| View reservations (no identity) | `/` | Shows times/dates but hides reserver names. | Yes |
| Access login/register | `/login`, `/register` | Forms are accessible. | Yes |
| **❌ Cannot do** | | | |
| Access booking form | `/reservation` | Redirects to `/login`. | Yes |
| Access profile page | `/profile` | Returns Not Found. | Yes |
| Check Session | `/api/session` | Returns 401 Unauthorized. | Yes |
| List users | `/api/users` | The list of users is visible as a guest. Major GDPR Violation. | No |
| List bookings | `/api/reservations` | The list of all reservations is visible as a guest. | No |
| Add a new resource | `/resources` | The guest can add a new resource. Broken Resource Management (Authorization Bypass). | No |

---

### 🧑‍💼 Reserver (Authenticated - User)
*Goal: Verify individual user actions and restricted admin access.*

| Action | Endpoint | Observation | Spec Match |
| :--- | :--- | :--- | :--- |
| **✅ Can do** | | | |
| Book a resource | `/reservation` | Accessible if user is > 15 years old. | Yes |
| View own profile | `/profile` | Shows user-specific data. | No |
| List bookings | `/api/reservations` | The list of all reservations is visible as a reserver role. | Yes |
| Check Session | `/api/session` | The reserver can check their username and role. | Yes |
| **❌ Cannot do** | | | |
| Modify resources | `/api/resources?id=:id` | A reserver can successfully modify a resource. | No |
| Delete resources | `/api/resources?id=:id` | A reserver cannot delete a resource. Error during deleting resource. | No |
| List users | `/api/users` | The list of users is visible as a reserver role. Major GDPR Violation. | No |
| Update reservations | `/reservation?id=:id` | A reserver can modify any booking. | No |

---

### 🧑‍💼🛡️ Administrator (Authenticated - High Privilege)
*Goal: Verify full system control.*

| Action | Endpoint | Observation | Spec Match |
| :--- | :--- | :--- | :--- |
| **✅ Can do** | | | |
| Modify resources | `/api/resources?id=:id` | An admin can successfully modify a resource. | Yes |
| Delete resources | `/api/resources?id=:id` | An admin cannot delete a resource. Error during deleting resource. | No |
| Delete a reserver | `/api/users/delete/:id` | Cannot remove a user from DB. | No |
| Update reservations | `/reservation?id=:id` | Can modify any booking. | Yes |
| Delete reservations | `/reservation?id=:id` | Can moddeleteify any booking. | Yes |
| **❌ Cannot do** | | | |
| Delete resources | `/api/resources?id=:id` | An admin cannot delete a resource. Error during deleting resource. (bug) | No |

---

## 3️⃣ Discovery & Vulnerability Findings

### 📂 Discovered Endpoints (Gobuster/ZAP)
* `/hidden-example-page`: [Describe what role can see this]
* `/api/v1/internal`: [Describe behavior]

### 🚨 Broken Access Control Issues (OWASP A01)
* **Vulnerability:** [e.g., IDOR on /profile?id=XXX]
* **Impact:** [e.g., Reserver A can see Reserver B's birthdate]
* **Status:** [Confirmed/Fixed]

---

## 4️⃣ Conclusion
[Briefly summarize if the system is GDPR compliant regarding Privacy by Design (PbD) principles based on your findings.]