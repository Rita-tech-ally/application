# Cronjob

| Feature | `grep CRON /var/log/syslog` | `journalctl -u cron` |
| --- | --- | --- |
| **Data Source** | Plain text file (`/var/log/syslog`) | Binary database (systemd-journald) |
| **Filtering Method** | Simple text match (finds the word "CRON") | Metadata match (finds the exact service) |
| **Accuracy** | Low (might catch unrelated messages) | High (only shows actual cron service logs) |
| **Historical Logs** | Only checks today's active file | Automatically searches all past logs |
| **Formatting** | Raw text, dumps everything at once | Paginated, colorized, easy to scroll |
| **Future Proofing** | Phased out on many modern distros | The standard for modern Linux |

---
**table format mein:**

| Feature | `/etc/crontab` | `/etc/cron.d/` |
| --- | --- | --- |
| **Type** | Single Text File | Directory (Folder) |
| **Management** | Sab kuch ek hi jagah hota hai | Alag-alag tasks ke liye alag files banai ja sakti hain |
| **Best For** | Core system OS tasks | Custom scripts aur third-party software installations |
| **Safety** | Edit karte waqt galti se dusre jobs delete ho sakte hain | Safe hai, kyunki aap sirf apni specific file edit ya delete karte hain|

---
**/etc/cron.hourly/ (Regular 'Cron' handle karta hai)** anacron ka design aisa hai ki uska minimum time period 1 din (1 day) hota hai. Yeh ghanto (hours) ya minutes ke hisaab se kaam nahi kar sakta.

**/etc/cron.daily/, weekly/, aur monthly/ ('Anacron' handle karta hai)** weekly, aur monthly jobs system ke liye bahut important hoti hain (jaise log clear karna, security updates check karna), isliye inhein anacron

**Short Summary:** OS aur software ke system-wide tasks /etc/ wale folders (crontab, cron.d, cron.daily) mein jaate hain, aur regular users ke custom tasks (jo crontab -e se bante hain) is /var/spool/cron/crontabs/ folder mein chup-chaap save hote hain.

---
**creating job for Specific user**

sudo crontab -u username -e **and** sudo crontab -u username -l 

---


# Requirement.txt

python3 --version,

pip3 --version

mkdir virtual

python3 -m venv venv, (creatind virtual envi)

source venv/bin/activate

pip freez > requirement.txt
pip list (list kran ke liya jo packag h eg pip list panada)

**for intalling package other system**

same step of starting 

python -m(module) venv .venv ( venv for vitual environment .venv creating Vi.. env.. current locaton)

deactive ( this is for deactive vir... env..)

---

#### UsesCases

###### Basic Use Cases (Rozmara ke kaam)
Environment Consistency: Har library ka exact version lock karna taaki code aapke dost ke PC par bhi bina error ke chale ("It works on my machine" problem solve).

**Easy Code Sharing:** Team ke sath heavy folders share karne ke bajaye sirf yeh choti text file share karna (pip install -r requirements.txt se sab ek sath install ho jata hai).

**Server Deployment:** Cloud servers (AWS, Heroku) ko batana ki aapke project ko run karne ke liye kaun-kaun se packages download karne hain.

**CI/CD Pipelines:**  Automated testing (GitHub Actions, Jenkins) ke time par test server ka environment jaldi aur automatically setup karna.

###### Advanced Use Cases (Professional & Enterprise level)

**Docker Caching (Fast Builds):** Docker mein is file ko code se pehle copy karke libraries cache karna, taaki code change karne par dobara install hone ka time bache.

**Security Audits:** Dependabot ya Snyk jaise tools dvara is file ko scan karwana, taaki purane ya risky packages par turant security alert mil sake.

**Offline Installation (Air-Gapped Systems):** Bina internet wale highly secure servers (Banks/Defense) ke liye kisi dusre PC par pip download -r se saare packages offline download karke pendrive se install karna.

**License Compliance (Legal Check):** Legal teams ke automated tools dvara yeh check karna ki kisi library ka license company ki policies (commercial use) ke khilaf toh nahi ha

---
##### Best pratices and depencies:

| **Rule (Niyam)**              | **Kahan Use Hota Hai (Target)**                    | **Syntax Example**       | **Kyun Zaroori Hai (Reason)**                                                                                  |
| ----------------------------- | -------------------------------------------------- | ------------------------ | -------------------------------------------------------------------------------------------------------------- |
| **1. Pin Exact Versions**     | Final Apps / Live Servers (**Production**)         | `Django==4.2.1`          | Live website par achanak naya update aane se app crash na ho.                                                  |
| **2. Controlled Flexibility** | Python Libraries (jo dusre developers use karenge) | `requests>=2.20.0`       | Dependency Hell se bachne ke liye, taaki alag-alag libraries clash na karein.                                  |
| **3. Semantic Versioning**    | Rule 2 ko safe banane ke liye                      | `requests>=2.5.0,<3.0.0` | Chote **bug/feature updates (Patch/Minor)** allow karein, lekin code todne wale **Major updates** ko rok dein. |

| **Topic**           | **Key Concept**             | **Important Point**                                                                                                                                              |
| ------------------- | --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **The Hidden Trap** | **Transitive Dependencies** | Sirf main library, jaise `Flask==2.0.1`, ko lock karna enough nahi hai. Uski **sub-dependencies** bhi version changes ki wajah se problem create kar sakti hain. |
| **The Solution**    | **Lock Files**              | Lock file main package ke saath **sub-dependencies ke exact versions** bhi save karti hai.                                                                       |
| **Security**        | **Package Hashes**          | Hashes package ki integrity verify karne mein help karte hain, taaki unexpected/corrupted package install na ho.                                                 |
| **Reproducibility** | **Same Environment**        | Lock file ki wajah se different machines ya future mein **same dependency versions** ke saath environment recreate kiya ja sakta hai.                            |
| **Tool 1**          | **pip-tools**               | `requirements.in` se locked `requirements.txt` generate karta hai.                                                                                               |
| **Tool 2**          | **Poetry**                  | Dependencies ko manage karta hai aur `poetry.lock` file generate karta hai.                                                                                      |
| **Tool 3**          | **Pipenv**                  | Dependencies ko manage karta hai aur `Pipfile.lock` generate karta hai.                    
|
pip ("Pip Installs Packages")
PyPI (Python Package Index)
---

# JQ

🔗 https://jqlang.github.io/jq/

jq is like sed for JSON data - you can use it to slice and filter and map and transform structured data with the same ease that sed, awk, grep and friends let you play with text.

---

| **API Type**             | **Kaam Kaise Karta Hai**                                                                              | **Kahan Use Hota Hai**                               |
| ------------------------ | ----------------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| **REST (RESTful API)**   | HTTP ke through request/response karta hai aur commonly **JSON** format mein data exchange karta hai. | Web apps, AWS/cloud services, aur microservices      |
| **SOAP**                 | Ek **strict protocol** hai jo mainly **XML** format use karta hai.                                    | Banking, enterprise applications, aur legacy systems |
| **GraphQL**              | Client ko **exactly required data** request karne deta hai, isliye extra data nahi aata.              | Mobile apps aur complex applications                 |
| **RPC (gRPC / XML-RPC)** | **Action/function-oriented** communication karta hai, jaise server par kisi function ko call karna.   | Internal servers aur high-performance microservices  |


---

**/usr/bin/jq** asal mein jq software ka main program hai.

**Executable Binary File:** Yeh ek compiled program hai. Yaani yeh aisi file hai jise computer seedha 'run' ya 'execute' karta hai.

**Short mein:** Jab bhi aap apne terminal mein jq command type karte hain, toh aapka Linux system chupchaap isi /usr/bin/jq file ke paas jata hai aur us program ko aapke liye chala deta hai.


**linux-amd64** ka matlab hai ki yeh 64-bit Linux OS (jaise Ubuntu ya CentOS) ke liye bani hui ek "ready-to-use" (pre-compiled) file hai.

#### Download hone ke baad kya karna hota hai?
Sirf is command ko chalane se jq install nahi hoga, wo bas ek normal file ki tarah aapke folder mein download ho jayega. Ise ek proper "command" banane ke liye aapko iske baad yeh 2 steps aur karne padte hain:

#### Step 1: Ise 'Executable' (chalne layaq) banayein
Download ki hui file par abhi run karne ki permission nahi hoti. Ise permission dene ke liye yeh command chalani padti hai:

#### Bash
chmod +x jq-linux-amd64
Step 2: Ise system ke main folder mein move karein (naam badal kar)
Taaki aap apne PC mein kahin se bhi sirf jq likh kar is tool ko chala sakein, ise /usr/local/bin/ folder mein move karna padta hai:

#### Bash
sudo mv jq-linux-amd64 /usr/local/bin/jq
Uske baad aap terminal mein seedha jq --version likh kar check kar sakte hain ki tool sahi se setup ho gaya hai.

# hash -r command Linux terminal (Bash shell) ki "Memory (Cache) ko clear/reset" karne ke liye use hoti hai.

Ise aur achhe se samajhne ke liye dekhte hain ki yeh kis problem ko solve karti hai:

**Background:** Terminal ki Memory (Hash Table)
Jab aap terminal mein koi command (jaise python, jq, ya nano) pehli baar type karte hain, toh Linux aapke system (aam taur par $PATH) mein usko dhundhta hai.
Jab wo mil jati hai, toh terminal agli baar apna time bachane ke liye us file ke location (rasta) ko yaad kar leta hai aur ek "Hash Table" (cache) mein save kar leta hai.

***Problem Kya Hoti Hai?***
Maan lijiye aapne system mein pehle se majood koi program (jaise jq) kisi nayi jagah par move kar diya (jaise pichle example mein humne /usr/local/bin/ mein kiya tha).

Ab agar aap terminal mein jq type karenge, toh terminal apni purani memory ke hisaab se use purani jagah par dhundhega, aur aapko ajeeb sa error dega:bash: /purana/rasta/jq: No such file or directory (Jabki file apne system mein nayi jagah par daal di hai).

**hash -r Kya Karta Hai?**
hash: Yeh command terminal ki uss memory (cache) ko dikhati hai jahan commands ke raste save hain.

-r (Reset / Remove): Yeh us poori memory (cache) ko delete kar deta hai.

**Nateeja (Result):**
Jab aap hash -r chalate hain, toh terminal sab kuch bhool jata hai. Ab agli baar jab aap jq (ya koi aur command) type karenge, toh terminal bina purani memory use kiye, naye sire se usko sahi jagah par dhundhega aur bina error ke run kar dega.

**Short Summary:** Agar aap kisi command/program ko ek folder se dusre folder mein move/update karte hain, aur terminal usko recognize nahi kar raha hai ya purana version dikha raha hai, toh hash -r chalane se problem solve ho jati hai.

. Normal Download (sudo apt install jq)Jab aap apt (Package Manager) ka use karte hain, toh aap system ko ek VIP service ka order dete hain. apt sirf file download nahi karta, wo background mein yeh saare kaam automatically (khud se) kar deta hai:

Wo sahi version dhundhta hai.File ko download karta hai.Usko automatically execute permission (chmod +x) de deta hai.File ko khud utha kar sahi folder (/usr/bin/jq) mein daal deta hai.Aapko kuch bhi manual nahi karna padta, sab kuch ek command mein ho jata hai. (Yeh bilkul Zomato se paka-pakaya khana order karne jaisa hai).

2. Specific Version (wget wala tareeqa)Jab aapko koi specific version (jaise 1.8) chahiye hota hai jo apt ke paas nahi hai, toh aap wget use karte hain. Lekin wget package manager nahi hai, yeh sirf ek downloader tool hai.Permission kyun deni padi (chmod)? Linux security ke mamle mein bahut strict hai.

Internet se download ki hui kisi bhi raw file ko Linux sirf ek "Text/Data file" maanta hai, virus/malware se bachne ke liye wo usko run karne se rokta hai. Isliye aapko OS ko batana pada ki "Yeh safe hai, ise software ki tarah run karne ki permission do (chmod +x)."Move kyun karna pada (mv)? wget ne file wahin download kardi jahan aap terminal mein khade the (jaise /home/ritu/). Agar aap usko move nahi karte, toh terminal ko kaise pata chalta ki jq kahan rakha hai? Usko system ke main software folder (/usr/local/bin/) mein daalna zaroori tha taaki aap kisi bhi folder se sirf jq type karein aur wo chal jaye. 

(Yeh kacha saaman kharid kar khud khana banane jaisa hai).


Quick Comparison (Aapke SOP ke hisaab se)FeatureMethod 1: apt install jq (Normal)

Method 2: wget (Specific Version)Downloadapt khud internet se uthata hai.Aapko wget aur URL dena padta hai.PermissionsAutomatically set ho jati hain.Manually chmod +x karna padta hai.LocationKhud /usr/bin/jq mein set hota hai.Manually /usr/local/bin/jq mein move (mv) karna padta hai.Best ForJab koi bhi stable version chalega.Jab exact wahi version chahiye jo latest GitHub par hai.

| Step | Command | Iska Use Kyun Kiya? (Reason) |
|---|---|---|
| 1️⃣ **Download** | `wget .../jq-linux-amd64` | `jq` ek **single ready-made binary file** hai. `wget` ka use karke internet se `jq` ki file download karke current folder mein save ki. |
| 2️⃣ **Permission** | `chmod +x jq-linux-amd64` | Download hone ke baad file ko **execute/run karne ki permission** deni hoti hai. `+x` Linux ko batata hai ki is file ko executable program ki tarah run kiya ja sakta hai. |
| 3️⃣ **Move & Rename** | `sudo mv jq-linux-amd64 /usr/local/bin/jq` | File ko `/usr/local/bin/` mein move karke `jq` naam diya. Isse har baar `./jq-linux-amd64` nahi likhna padega. Ab terminal mein **kahin se bhi `jq`** command chala sakte hain. |
| 4️⃣ **Reset Cache** | `hash -r` | Shell kabhi-kabhi commands ki **purani location cache** karke rakhta hai. `hash -r` cache ko refresh karta hai, taaki shell naye `jq` command ko turant identify kar sake. |



