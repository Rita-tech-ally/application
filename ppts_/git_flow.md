# Git Flow and GitHub, GitLab, Bitbucket

### 1. Git Flow (The Branching Strategy)

Git Flow ek bahut hi structured aur professional "Branching Model" hai. Iska main rule ye hai ki hum directly production code ko touch nahi karte. Code ko safe rakhne ke liye isme **5 specific branches** ka use hota hai:

- **`main` (ya `master`):** Ye production branch hai. Iska code live servers par chalta hai. Isme hamesha 100% stable aur tested code hota hai.
- **`develop`:** Ye hamari main "Integration" branch hai. Aage aane wale jitne bhi naye features hain, wo sab pehle isme aakar merge hote hain.
- **`feature`:** Jab bhi kisi developer ko naya task milta hai (e.g., "Add Login Button"), toh wo `develop` se ek `feature/login` branch banata hai. Kaam pura hone par ise wapas `develop` me merge kar diya jata hai.
- **`release`:** Jab `develop` me kaafi naye features aa jate hain, toh hum testing ke liye ek `release` branch banate hain. Yahan minor bug fixes hote hain. Pass hone ke baad ye `main` me merge ho jati hai.
- **`hotfix`:** Agar live production (`main` branch) me achanak koi bada bug (error) aa jaye, toh seedha `main` se `hotfix` branch nikalte hain. Bug fix karke ise wapas `main` aur `develop` dono me merge kar dete hain.

### 2. GitHub, GitLab, aur Bitbucket Kya Hain?

Sabse pehle ek chota sa concept clear rakhna zaroori hai:

- **Git** ek local software/tool hai (jo aapke laptop me code history track karta hai).
- **GitHub, GitLab, aur Bitbucket** online hosting platforms hain. Ye "Cloud Servers" hain jahan aap apna Git wala code upload (push) karte hain taaki aapki poori team ek sath us par kaam kar sake.

### 3. In Teeno Me Asli Difference Kya Hai?

Halanki teeno ka main kaam code host karna hai, par DevOps industry me inka Use-Case bilkul alag hai:

| **Feature** | **GitHub** | **GitLab** | **Bitbucket** |
|---|---|---|---|
| **Company Owner** | Microsoft | GitLab Inc. | Atlassian |
| **Sabse Zyada Kahan Use Hota Hai?** | Open-source projects aur public repositories ke liye (Duniya ka sabse bada platform). | **DevOps aur CI/CD** ke liye. Ye ek "All-in-One" DevOps tool hai. | Un companies me jo **Jira aur Confluence** ka bahut zyada use karti hain. |
| **CI/CD Integration** | GitHub Actions use karta hai. | **GitLab CI/CD** (Jo industry me Jenkins ke baad sabse powerful mana jata hai). | Bitbucket Pipelines use karta hai. |
| **Pull Request (PR) Term** | Ise **Pull Request (PR)** bolte hain. | Ise **Merge Request (MR)** bolte hain. | Ise **Pull Request (PR)** bolte hain. |
| **Self-Hosted (On-Premise)** | Enterprise version mehnga hai. | Free tier me bhi apne khud ke servers (EC2) par easily install ho jata hai. | Server version available hai, par Atlassian ise dheere-dheere band kar raha hai. |

# Git Flow and GitHub, GitLab, Bitbucket

### 1. Git Flow (The Branching Strategy)

Git Flow ek bahut hi structured aur professional "Branching Model" hai. Iska main rule ye hai ki hum directly production code ko touch nahi karte. Code ko safe rakhne ke liye isme **5 specific branches** ka use hota hai:

- **`main` (ya `master`):** Ye production branch hai. Iska code live servers par chalta hai. Isme hamesha 100% stable aur tested code hota hai.
- **`develop`:** Ye hamari main "Integration" branch hai. Aage aane wale jitne bhi naye features hain, wo sab pehle isme aakar merge hote hain.
- **`feature`:** Jab bhi kisi developer ko naya task milta hai (e.g., "Add Login Button"), toh wo `develop` se ek `feature/login` branch banata hai. Kaam pura hone par ise wapas `develop` me merge kar diya jata hai.
- **`release`:** Jab `develop` me kaafi naye features aa jate hain, toh hum testing ke liye ek `release` branch banate hain. Yahan minor bug fixes hote hain. Pass hone ke baad ye `main` me merge ho jati hai.
- **`hotfix`:** Agar live production (`main` branch) me achanak koi bada bug (error) aa jaye, toh seedha `main` se `hotfix` branch nikalte hain. Bug fix karke ise wapas `main` aur `develop` dono me merge kar dete hain.

### 2. GitHub, GitLab, aur Bitbucket Kya Hain?

Sabse pehle ek chota sa concept clear rakhna zaroori hai:

- **Git** ek local software/tool hai (jo aapke laptop me code history track karta hai).
- **GitHub, GitLab, aur Bitbucket** online hosting platforms hain. Ye "Cloud Servers" hain jahan aap apna Git wala code upload (push) karte hain taaki aapki poori team ek sath us par kaam kar sake.

### 3. In Teeno Me Asli Difference Kya Hai?

Halanki teeno ka main kaam code host karna hai, par DevOps industry me inka Use-Case bilkul alag hai:

| **Feature** | **GitHub** | **GitLab** | **Bitbucket** |
|---|---|---|---|
| **Company Owner** | Microsoft | GitLab Inc. | Atlassian |
| **Sabse Zyada Kahan Use Hota Hai?** | Open-source projects aur public repositories ke liye (Duniya ka sabse bada platform). | **DevOps aur CI/CD** ke liye. Ye ek "All-in-One" DevOps tool hai. | Un companies me jo **Jira aur Confluence** ka bahut zyada use karti hain. |
| **CI/CD Integration** | GitHub Actions use karta hai. | **GitLab CI/CD** (Jo industry me Jenkins ke baad sabse powerful mana jata hai). | Bitbucket Pipelines use karta hai. |
| **Pull Request (PR) Term** | Ise **Pull Request (PR)** bolte hain. | Ise **Merge Request (MR)** bolte hain. | Ise **Pull Request (PR)** bolte hain. |
| **Self-Hosted (On-Premise)** | Enterprise version mehnga hai. | Free tier me bhi apne khud ke servers (EC2) par easily install ho jata hai. | Server version available hai, par Atlassian ise dheere-dheere band kar raha hai. |

# Branching Strategies in DevOps

DevOps industry me code ko safe rakhne aur team ke beech conflicts (takrav) ko rokne ke liye alag-alag **Branching Strategies** use hoti hain. Ek Senior DevOps Engineer ke taur par, aapko in chaaro ka difference ekdum clear hona chahiye.

Aaiye inko simple aur professional tareeke se samajhte hain:

### 1. Feature Branch Flow (The Simplest Approach)

Ye sabse basic aur popular model hai. Iska main rule ye hai ki `main` branch me seedha koi code nahi likhega.

- **Kaise Kaam Karta Hai:** Jab bhi kisi developer ko naya task milta hai (jaise login page banana), toh wo `main` se ek nayi branch nikalta hai (`feature-login`). Kaam pura hone par ek PR (Pull Request) banata hai. Code review hota hai, pipeline chalti hai, aur agar sab theek hai, toh code wapas `main` me merge ho jata hai.
- **Fayda:** Code isolate (alag) rehta hai. Chhoti teams aur fast CI/CD pipelines ke liye ye best hai.
- **Kahan Use Hota Hai:** Agile projects jahan din me kayi baar code live hota hai.

### 2. Git Flow (The Strict & Traditional Approach)

Ye kafi complex aur strict model hai. Isme code sidha live hone ke bajaye alag-alag stages se guzarta hai.

- **Kaise Kaam Karta Hai:** Isme 5 specific branches hoti hain:
  1. **`main`:** Sirf live/production code.
  2. **`develop`:** Saare developers ka code yahan aakar integrate hota hai.
  3. **`feature`:** `develop` se nikalkar wapas `develop` me merge hoti hai.
  4. **`release`:** Jab `develop` me kaafi code aa jata hai, toh testing ke liye `release` branch banti hai. Test pass hone par ye `main` aur `develop` dono me merge hoti hai.
  5. **`hotfix`:** Agar live server par koi bug aa jaye, toh `main` se nikal kar theek karte hain aur wapas `main` + `develop` me bhejte hain.
- **Fayda/Nuksan:** Bahut safe hai, par bahut slow hai. Isme "Merge Conflicts" bahut zyada aate hain.
- **Kahan Use Hota Hai:** Mobile apps ya enterprise software jinka update mahine me sirf ek baar aata ho (Scheduled Releases).

### 3. Environment Branch Flow (The Server-Matched Approach)

Is model me branches seedha aapke servers (environments) ko match karti hain.

- **Kaise Kaam Karta Hai:** Isme normally 3 branches hoti hain: `dev`, `staging` (testing), aur `prod` (live). Developer pehle code `dev` branch me merge karta hai, jisse Dev server update hota hai. QA team test karti hai. Phir code `dev` se `staging` branch me merge hota hai, aur wahan se final `prod` branch me.
- **Fayda:** DevOps pipeline banana sabse aasan hota hai (Branch = Server).
- **Nuksan:** Agar `staging` me 5 features pade hain, par business kehta hai ki sirf 2 ko live karo, toh unhe alag karke `prod` me bhejna bahut mushkil ho jata hai.

### 4. GitLab Flow (The "Upstream-First" Modern Approach)

Ye Feature Branch aur Environment Branch ka ek smart combination hai jo DevOps/CI-CD ke liye banaya gaya hai. Iska sabse bada rule hai **"Upstream First"** (Code hamesha pehle `main` me aayega, uske baad hi aage jayega).

-
