# Notification Worker (README / Presentation Script)

### 1. Introduction (Notification Worker kya hai)

Notification Worker koi aam REST API nahi hai, ye ek **background scheduled worker** hai. Iska main kaam hai scheduled time (jaise mahine ki shuruwat me ya har ghante) par background me chalna aur employees ko unki "Salary Slip" ke notifications email ke zariye bhejna. Ye **Python** me likha gaya hai.

### 2. Tech Stack Explanation (Point-by-Point)

- **Python + Schedule Library:** Isme koi web server (jaise Flask ya Gin) nahi hai. Ye ek continuous chalne wali script hai jo `schedule` module use karti hai taaki fixed interval par function execute ho sake.
- **Elasticsearch Integration:** Ye kisko mail bhejna hai uska data (jaise Employee Emails) Elasticsearch se fetch karta hai (`elasticsearch` python client ke through). Ye data write nahi karta, sirf **read** karta hai.
- **SMTP Email Library:** Emails send karne ke liye ye `emails` library aur external SMTP server (jaise SendGrid, Mailchimp ya custom SMTP) ka use karta hai.

### Step-by-Step Explanation

Notification worker aam APIs se alag kaam karta hai (Isme client request nahi aati):

1. **Application start hoti hai** aur config variables (`config.yaml` se SMTP aur ES details) read karti hai.
2. Application apne mode ke hisaab se chalti hai:
   - **External mode:** Ek baar sabko email bhejo aur stop ho jao.
   - **Scheduled mode:** Background loop me chalti rahegi aur fix time par trigger hogi.
3. **Trigger hone par:** Elasticsearch pe ek `match_all` query chalti hai jo saare employees ka data utha lati hai.
4. Loop chalta hai aur har employee ke `email_id` ko `send_mail()` function me pass kiya jata hai.
5. Ye function SMTP credentials use karke SMTP server se connect hota hai aur actual mail send kar deta hai.

### Kyun Ye Approach Use Ki Gayi

- Emails bhejna ek slow aur time-taking task hota hai. Agar is kaam ko Employee ya Salary API ke andar daal dete, to user ka response delay ho jata. Ek alag asynchronous background worker banane se baaki APIs fast aur independent rehti hain.

## Notification Worker ka Baaki Microservices ke Saath Connection

Ye ek bahut achha aur logical sawal hai! Agar aap dhyan se dekhein, toh **Notification Worker** kisi bhi doosri API (Employee, Salary, Attendance) se seedha connect nahi karta. Microservices me is approach ko **Decoupled Architecture (Asynchronous Communication)** kehte hain.

Chaliye is poore flow ko 3 steps me samajhte hain ki data kaise aata hai aur email kaise jata hai:

### 1. Elasticsearch me Data Kaise Aayega? (The "Dual-Write" Pattern)

Agar aap `notification_api.py` ka code dekhein, toh wo sirf Elasticsearch se data **read** kar raha hai (`es_client.search(...)`). Elasticsearch me data **Employee API (Go)** daal rahi hai.

Is architecture me **Dual-Write Pattern** use hota hai:

- Jab frontend se naya employee add hota hai, **Employee API (Go)** sabse pehle us data ko apne primary database (**ScyllaDB**) me save karti hai.
- ScyllaDB me save hone ke turant baad, usi Go API ke code me agla step chalta hai, jo same data ko seedha **Elasticsearch** ke `employee-management` index me push (Index) kar deta hai. *(Note: Is kaam ke liye Logstash ya kisi aur tool ka use nahi hota, ye Go API ke code ke andar hi likha hota hai).*

### 2. Email Kaise Send Hoga?

Email send karne ka logic puri tarah se `notification_api.py` me likha hai:

1. **Fetch Emails:** Script Elasticsearch par ek query marti hai aur saare employees ka data utha lati hai.
2. **Looping:** Phir wo ek `for` loop chalati hai jisme se wo `email_id` nikal leti hai.
3. **SMTP Connection:** Python ki `emails` library use karke, script `config.yaml` se SMTP server ka username/password aur port uthati hai.
4. **Sending Mail:** Har email ID par ek HTML message bhej diya jata hai.

### 3. Baaki Services Ke Saath Connection Kaise Hota Hai?

Notification Worker baaki services se **directly connect hota hi nahi hai!** Ye is architecture ki sabse badi khoobsurti hai.

- **Independent Execution:** Is script ko kisi API request ka intezaar nahi hota. Isme Python ka `schedule` module use hua hai. Ye background me jaagta hai, Elasticsearch se data uthata hai, mails bhejta hai, aur wapas so jata hai.
- **Fayda (Advantage):** Agar kal ko SMTP server down ho jaye ya 10,000 employees ko mail bhejte waqt server slow ho jaye, toh iska asar **Employee API ya Frontend par bilkul nahi padega**.

## Data Synchronization Pipeline: ScyllaDB to Elasticsearch

Agar Employee API apna data **ScyllaDB** me save kar rahi hai, aur Notification Worker **Elasticsearch** se padh raha hai, toh data wahan tak pahuncha kaise?

### The "Dual-Write" Approach (No Logstash needed for Business Data)

ScyllaDB ek bada Godaam (Warehouse) hai jahan saara original data safe rakha jata hai. Aur Elasticsearch ek fast Telephone Directory hai, jahan se email IDs jaldi dhoondhi ja sakti hain.

### Data Flow ke 3 Steps

1. **Frontend Request:** Jab frontend se naya employee add hota hai, wo request `/employee/create` endpoint par (Go API ko) aati hai.
2. **First Write (ScyllaDB):** Go API us data ko chup-chap **ScyllaDB** (Godaam) me permanently save kar deti hai.
3. **Second Write (Elasticsearch):** Turant baad, Go API ek HTTP request ke through wahi naya data **Elasticsearch** (Directory) ke `employee-management` index me push kar deti hai. *(Ab data Notification Worker ke read karne ke liye taiyar hai).*

### Fayda kya hua iska?

ScyllaDB data ko permanently safe rakhne ke liye best hai, lekin usme se hazaron emails search karna slow ho sakta hai. Elasticsearch specially **fast searching** ke liye hi bana hai. Data ko dual-write karke Elasticsearch me rakhne se Notification Worker milliseconds me hazaron emails nikal kar mails bhej sakta hai, bina main Employee database ko slow kiye.

### 🚨 Important Clarification: Phir Logstash Kaam Kahan Aata Hai?

Aksar log confuse ho jate hain ki Logstash data sync kar raha hai. **OT-Microservices me Logstash ka Notification ya Salary business logic se koi lena-dena nahi hai.**

Logstash purely **Centralized Logging (System Monitoring)** ke liye use hota hai:

1. **Asli Kaam:** Jab aapki Go, Java, ya Python API me koi code error aata hai, server crash hota hai, ya koi user login karta hai, toh jo "Logs" (Terminal messages) generate hote hain, Logstash unhe ikattha karta hai.
2. **Kahan Bhejta Hai:** Logstash un system logs ko Elasticsearch me alag index me save karta hai, taaki DevOps team **Kibana** dashboard par un errors ko dekh sake.

**Simple Flow (Business Data):** `Frontend → Go API → (Dual-Write) → ScyllaDB & Elasticsearch → Notification Worker → Email`

**Simple Flow (System Logs/Errors):** `Go/Java/Python API (Console Logs) → Logstash → Elasticsearch → Kibana (Dashboard)`
