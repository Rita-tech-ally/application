# Git Hooks – Definition, Types, and Implementation

## What Are Git Hooks?

**Git Hooks** ek tarah ke automated scripts hote hain jo kisi specific Git action, jaise commit, push, ya merge, ke time par automatically execute hote hain.

Isko ek **"Security Guard" ya "Quality Checker"** ki tarah samjhiye. Jab aap `git commit` command run karte hain, toh Git commit complete karne se pehle kuch hooks execute karta hai. Agar hook mein koi error aata hai, toh woh commit ko rok sakta hai.

## Commit Hook Kya Hota Hai?

Commit hook basically aapke code ka ek **"Automated Security Guard"** hota hai.

Jab aap `git commit` type karte hain, toh Git commit create karne se pehle configured scripts run kar sakta hai. In scripts ko Git Hooks kehte hain.

### Real-Life Example: Airport Security

- **Aapka Code:** Passenger hai.
- **`git commit` command:** Flight mein board karne ki koshish hai.
- **Commit Hook:** Airport ka Metal Detector (Security Check) hai.

Agar code mein koi issue hai, jaise tests fail ho rahe hain ya AWS password/secret key accidentally add ho gayi hai, toh configured hook commit ko rok sakta hai. Terminal par error message dikh sakta hai, jo batata hai ki pehle issue fix karein.

**Ek line mein:** Git Hooks automated scripts hain jo code ko commit ya push karne jaise Git actions ke dauran checks aur tasks perform karte hain.

## Types of Commit Hooks

Commit process ko control karne ke liye Git chaar important commit-related hooks provide karta hai, jo is order mein execute ho sakte hain:

### 1. `pre-commit`

- **Kab chalta hai:** Commit create hone se pehle aur commit message editor open hone se pehle.
- **Kyun use hota hai:** Code quality checks perform karne ke liye.
- **Examples:**
  - Linting checks, jaise ESLint.
  - Trailing whitespace check karna.
  - Chhote unit tests run karna.
  - Secrets scan karna.

Agar hook non-zero exit status return karta hai, toh commit abort ho jata hai.

### 2. `prepare-commit-msg`

- **Kab chalta hai:** Commit message editor open hone se pehle.
- **Kyun use hota hai:** Commit message ko automatically prepare ya modify karne ke liye.
- **Example:** Branch name se Jira ticket number nikal kar commit message mein `[PROJ-123]` add karna.

### 3. `commit-msg`

- **Kab chalta hai:** Commit message prepare karne ke baad, lekin commit create hone se pehle.
- **Kyun use hota hai:** Commit message validate karne ke liye.
- **Examples:**
  - Message mein minimum 10 characters check karna.
  - Message `feat:` ya `fix:` se start ho raha hai ya nahi, check karna.

Agar validation fail hoti hai, toh commit abort ho sakta hai.

### 4. `post-commit`

- **Kab chalta hai:** Commit successfully create hone ke baad.
- **Kyun use hota hai:** Commit ke baad koi task perform karne ke liye.
- **Examples:**
  - Local notification bhejna.
  - Logging ya reporting karna.
  - Kisi custom script ko trigger karna.

**Important:** `post-commit` hook commit ko abort nahi kar sakta, kyunki commit pehle hi create ho chuka hota hai.

## How to Use a Git Hook (Step-by-Step)

Git hooks ki sample files aam taur par repository ke `.git/hooks` folder mein hoti hain. Sample files ko rename karna zaroori nahi hai; aap sample file ko reference ke liye use karke usi naam se executable hook bana sakte hain.

### Step 1: Hooks Folder Open Karein

Apne project ke root folder mein jayen:

```bash
cd .git/hooks
```

Yahan aapko `pre-commit.sample` jaisi sample files dikh sakti hain.

### Step 2: Pre-Commit Hook File Banayein

```bash
mv pre-commit.sample pre-commit
```

Agar file pehle se maujood nahi hai ya aap sample file ko preserve karna chahte hain, toh nayi file banayein:

```bash
nano pre-commit
```

### Step 3: Script Likhein

`pre-commit` file mein ye Bash script add karein:

```bash
#!/bin/sh

echo "Checking code before commit..."

# Example: Run lint checks
# npm run lint

# Example: Run Python tests
# pytest

exit 0
```

Yahan `exit 0` ka matlab hai ki script successful hui aur commit process aage badh sakta hai. Real project mein linting ya testing commands ko uncomment karke configure karein.

### Step 4: Executable Permission Dein

```bash
chmod +x pre-commit
```

Ab jab aap `git commit` run karenge, toh Git is hook ko execute karega.

### Step 5: Hook Test Karein

Koi file change karke stage karein:

```bash
git add .
git commit -m "test: verify pre-commit hook"
```

Aapko terminal mein ye message dikhna chahiye:

```text
Checking code before commit...
```

## Industry/DevOps Pro Tip

Default `.git/hooks` folder Git repository mein push nahi hota. Isliye aapke laptop par banaya gaya hook team ke doosre developers ke laptops par automatically nahi aayega.

Is problem ko solve karne ke liye teams hook-management tools use karti hain:

- **Husky:** JavaScript/Node.js projects mein Git Hooks manage karne ke liye.
- **pre-commit framework:** Multiple languages ke projects mein hooks configure aur run karne ke liye.

In tools ki configuration files repository mein commit ki ja sakti hain, jisse team ke developers same checks configure kar sakte hain.

## Quick Revision

| Hook | Kab chalta hai? | Main Use |
|---|---|---|
| `pre-commit` | Commit message editor se pehle | Linting, tests, secret checks |
| `prepare-commit-msg` | Message editor se pehle | Commit message prepare/modify karna |
| `commit-msg` | Message prepare hone ke baad, commit se pehle | Message validate karna |
| `post-commit` | Commit create hone ke baad | Notifications aur logging |


# Git Hooks: DevOps Perspective — Pre-Commit vs CI/CD Pipeline

Haan, bilkul! Ek `pre-commit` hook ke andar aap multiple checks add kar sakti hain. Kyunki ye ek executable script hoti hai, ismein aap terminal commands chala sakti hain.

Lekin as a DevOps Engineer, aapko ye samajhna zaroori hai ki **kaunsa task `pre-commit` hook mein rakhna chahiye aur kaunsa CI/CD pipeline (jaise Jenkins ya GitLab CI) mein chalana chahiye.**

## 1. Linting & Formatting — Perfect for Pre-Commit

Ye Git Hooks ka sabse common use case hai. Code commit hone se pehle formatting check karna aur common coding mistakes pakadna iska main purpose hai.

**Examples:**
- Go API ke liye `golangci-lint`
- Python ke liye `flake8` ya `black`
- Ansible playbooks ke liye `ansible-lint`

Agar configured check fail hota hai, toh hook non-zero exit status return karke commit ko rok sakta hai.

## 2. Security Scanners — Lightweight Checks in Pre-Commit

Security checks ko lightweight aur heavy checks mein divide kar sakte hain.

### Pre-Commit Mein Kya Karein?

Secret scanners, jaise `gitleaks`, ko pre-commit hook mein configure kiya ja sakta hai. Ye developers ko AWS keys, database passwords ya API tokens accidentally commit karne se rokne mein help karta hai.

### CI/CD Mein Kya Karein?

Heavy security checks, jaise comprehensive SAST aur DAST scans, ko aam taur par CI/CD pipeline mein chalana behtar hota hai.

Examples:
- SonarQube analysis
- Comprehensive SAST scans
- DAST scans against a running application

Ye checks zyada time aur resources le sakte hain. Isliye inhe Jenkins ya GitLab CI pipeline mein run karna practical hota hai.

**Important:** Local hooks ko bypass kiya ja sakta hai. Isliye security checks ke liye CI/CD pipeline mein bhi validation honi chahiye.

## 3. Unit Tests — Fast Tests in Pre-Commit

- Agar unit tests chhote hain aur kuch seconds mein complete ho jaate hain, toh unhe pre-commit hook mein rakhna useful ho sakta hai.
- Examples: selected Go tests ke liye `go test` ya fast Python tests ke liye `pytest`.
- Agar tests ko 2–3 minutes ya usse zyada lagte hain, toh har commit par unhe run karna developers ke workflow ko slow kar sakta hai.

Aise cases mein fast checks local hook mein aur complete test suite CI/CD pipeline mein run karna behtar hota hai.

## 4. Logging & Notifications — Post-Commit or CI/CD

### Logging

Local commit-related information log karne ke liye hook use kiya ja sakta hai.

Example:

```bash
echo "Commit at $(date)" >> commit.log
```

Lekin `commit.log` ko repository mein add karne se har commit par extra changes aa sakte hain. Isliye local logs ko aam taur par repository ke bahar rakhna behtar hai.

### Notifications

Commit hone se pehle Slack ya email notification bhejna generally avoid karna chahiye. Agar network ya notification service fail ho jaaye, toh commit unnecessarily block ho sakta hai.

**Sahi approach:**
- Local notification ke liye `post-commit` hook use kiya ja sakta hai.
- Team-wide notifications ke liye CI/CD pipeline use karna zyada reliable hota hai.

Dhyan rahe, `post-commit` hook commit create hone ke baad chalta hai. Isliye ye commit ko undo ya abort nahi kar sakta.

## Quick Comparison Table

| Task | Pre-Commit Hook | CI/CD Pipeline |
|---|---|---|
| Linting & Formatting | Recommended | Recommended |
| Secret Scanning | Fast checks | Additional validation |
| Fast Unit Tests | Suitable | Recommended |
| Full Test Suite | Usually avoid | Recommended |
| SonarQube Analysis | Usually avoid | Recommended |
| Heavy SAST/DAST Scans | Usually avoid | Recommended |
| Local Logging | Possible | Pipeline logs |
| Team Notifications | Usually avoid | Recommended |

## DevOps Best Practice

Ek effective workflow mein **local Git Hooks fast feedback dete hain, jabki CI/CD pipeline comprehensive validation karti hai.**

- **Pre-commit:** Fast linting, formatting aur secret checks.
- **Pre-push (optional):** Kuch additional local checks.
- **CI/CD pipeline:** Full tests, SonarQube analysis, security scans aur deployment validations.
- **Post-commit:** Optional local logging ya notification.

Is tarah developers ko jaldi feedback milta hai aur CI/CD pipeline code ko merge ya deploy karne se pehle additional checks perform kar sakti hai.
