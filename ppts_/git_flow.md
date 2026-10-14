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
