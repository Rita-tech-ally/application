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

