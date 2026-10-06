# 3. Notification Worker (README / Presentation Script)

### 1. Introduction (Notification Worker kya hai)

Notification Worker koi aam REST API nahi hai, ye ek **background scheduled worker** hai. Iska main kaam hai scheduled time (jaise mahine ki shuruwat me) par background me chalna aur employees ko unki "Salary Slip" ke notifications email ke zariye bhejna. Ye **Python** me likha gaya hai.

### 2. Tech Stack Explanation (Point-by-Point)

- **Python + Schedule Library:** Isme koi web server (jaise Flask ya Gin) nahi hai. Ye ek continuous chalne wali script hai jo `schedule` module use karti hai taaki fixed interval (every hour / day) par function execute ho sake.
- **Elasticsearch Integration:** Ye kisko mail bhejna hai uska data (jaise Employee Emails) Elasticsearch se fetch karta hai (`elasticsearch` python client ke through).
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

### 1. Elasticsearch me Data Kaise Aayega?

Agar aap `notification_api.py` ka code dekhein, toh wo sirf Elasticsearch se data **read** kar raha hai (`es_client.search(...)`). Lekin Elasticsearch me data daal kaun raha hai?

Microservices me iske liye aamtaur par do tarike use hote hain:

- **Change Data Capture (CDC) / Logstash:** Jab **Employee API** naya employee ScyllaDB me save karti hai, toh ek background tool (jaise Logstash ya Kafka Connect) us database ko monitor karta hai aur automatically wo naya data Elasticsearch ke `employee-management` index me push kar deta hai.
- **Frontend se Direct API Call:** Aapke `EmployeeForm.js` me ek code likha hai jo ek request `/employee/create` ko aur doosri `/notification/send` ko bhejta hai. Yani frontend bhi directly ek notification service ko data bhej sakta hai jo use Elasticsearch ya message queue me daal de.

*(Kyunki Elasticsearch search ke liye bahut fast hai, isliye Notification Worker seedha ScyllaDB par load daalne ke bajaye Elasticsearch se email IDs nikalta hai.)*

### 2. Email Kaise Send Hoga?

Email send karne ka logic puri tarah se `notification_api.py` me likha hai:

1. **Fetch Emails:** Script Elasticsearch par ek query marti hai aur saare employees ka data utha lati hai.
2. **Looping:** Phir wo ek `for` loop chalati hai: `for data in result["hits"]["hits"]:` jisme se wo `email_id` nikal leti hai.
3. **SMTP Connection:** Python ki `emails` library use karke, script `config.yaml` se SMTP server (jaise Gmail, SendGrid, Amazon SES, ya Mailtrap) ka username/password aur port uthati hai.
4. **Sending Mail:** Har email ID par ek HTML message (*"Your salary slip is generated please check"*) bhej diya jata hai.

### 3. Baaki Services Ke Saath Connection Kaise Hota Hai?

Notification Worker baaki services se **directly connect hota hi nahi hai!** Ye is architecture ki sabse badi khoobsurti hai.

- **Independent Execution:** Is script ko kisi API request ka intezaar nahi hota. Isme Python ka `schedule` module use hua hai: `schedule.every().hour.do(...)`. Ye har ghante (ya month-end par) apne aap background me jaagta hai, Elasticsearch se data uthata hai, mails bhejta hai, aur wapas so jata hai.
- **Fayda (Advantage):** Agar kal ko SMTP server down ho jaye ya 10,000 employees ko mail bhejte waqt server slow ho jaye, toh iska asar **Employee API ya Frontend par bilkul nahi padega**. User aaram se UI use kar payega kyunki mail bhejne ka heavy kaam background me alag se ho raha hai.

## Data Synchronization Pipeline: ScyllaDB to Elasticsearch

Aapne bilkul architecture ka sabse **main catch** pakda hai!

Agar Employee API apna data **ScyllaDB** me save kar rahi hai, aur Notification Worker **Elasticsearch** se padh raha hai, toh data wahan tak pahuncha kaise? Kyunki dono ka database toh alag hai.

Is gap ko fill karne ke liye OT-Microservices jaise enterprise architectures me ek **"Data Synchronization Pipeline"** (aamtaur par **Logstash** ya Kafka) ka use kiya jata hai. Chaliye isko ek simple example se samajhte hain:

### The "Bridge" (Logstash / CDC)

Aap aise samjhiye ki **ScyllaDB** ek bada Godaam (Warehouse) hai jahan saara original data safe rakha jata hai. Aur **Elasticsearch** ek fast Telephone Directory hai, jahan se email IDs jaldi dhoondhi ja sakti hain.

Ab in dono ke beech me ek Helper baitha hota hai jise **Logstash** (ya Change Data Capture - CDC tool) kehte hain.

### Data Flow ke 4 Steps

1. **Employee Add Hota Hai:** Jab frontend se naya employee add hota hai, Employee API usey chup-chap **ScyllaDB** (Godaam) me save kar deti hai.


<img width="937" height="668" alt="Screenshot from 2026-10-06 18-39-01" src="https://github.com/user-attachments/assets/7eee6623-e89c-4de9-aaf9-bd7907e4ad05" />

2. **Helper (Logstash) React Karta Hai:** Logstash 24/7 ScyllaDB ko monitor karta rehta hai. Jaise hi wahan koi naya employee aata hai ya kisi ki salary update hoti hai, Logstash turant us naye data ki ek copy uthata hai.
3. **Elasticsearch Me Copy Save Hoti Hai:** Logstash us data ko ek proper format me convert karke **Elasticsearch** (Directory) ke `employee-management` index me push kar deta hai.
4. **Notification Worker Kaam Karta Hai:** Jab Notification Worker (Python script) apne scheduled time par jaagta hai, toh wo seedha Elasticsearch ki fast directory se saare naye emails nikalta hai aur SMTP ke through mail bhej deta hai.

### Frontend me Ek Aur "Shortcut" Hai

Agar aapne apne frontend ka code (`EmployeeForm.js`) dhyan se dekha ho, toh wahan ek interesting cheez hai:

```javascript
fetch('/employee/create', ...) // ScyllaDB me save karne ke liye
fetch('/notification/send', ...) // Notification ke liye shortcut
```

Iska matlab hai ki architecture me **Instant Welcome Emails** bhejne ke liye Frontend direct ek API request bhi bhej raha hai. Lekin jo **Monthly Salary Slips** jaisa bulk (bada) kaam hai, wo background me Python Worker + Elasticsearch ke combo se hi hota hai.

### Fayda kya hua iska?

ScyllaDB data ko permanently safe rakhne ke liye best hai, lekin usme se hazaron emails search karna slow ho sakta hai. Elasticsearch specially **fast searching** ke liye hi bana hai. Data ko sync karke Elasticsearch me rakhne se Notification Worker milliseconds me hazaron emails nikal kar mails bhej sakta hai, bina main Employee database ko slow kiye.
