# Database Selection in OT-Microservices Architecture

Ye ek **"System Design"** round ka sabse classic aur high-level sawal hai! Interviewer ye check karna chahta hai ki kya aapne aankh band karke in databases ka use kiya hai, ya aapko inke peeche ka "Kyun (Why)" pata hai.

Agar aap ye justification dengi, toh interviewer ko clear ho jayega ki aapki Architecture level ki soch hai. Aaiye isko ekdum logical way me samajhte hain:

### 1. PostgreSQL (For Salary & Attendance APIs)

- **Humne Kyun Chuna?** Salary aur Attendance ka data highly relational (tables, rows) hota hai. Isme paise ka len-den aur complex calculations hoti hain. PostgreSQL 100% **ACID compliant** hai (yani data kabhi corrupt ya aada-adhura save nahi hota) aur complex JOIN queries me sabse powerful open-source database hai.
- **Alternative Kya Tha?** **MySQL** ya **Oracle DB**
- **Humne Inhe Kyun Reject Kiya?** Oracle DB enterprise level par bahut mehnga (expensive) hai, jabki Postgres free aur open-source hai. MySQL bhi achha hai, lekin PostgreSQL data integrity, strictness aur complex SQL features me MySQL se thoda zyada advanced aur robust mana jata hai.

### 2. ScyllaDB (For Employee API)

- **Humne Kyun Chuna?** Naye employees ka data high-traffic wala hota hai jise milliseconds me load hona chahiye. ScyllaDB C++ me bana hai aur **Masterless Architecture** par kaam karta hai (yani isme koi ek main server nahi hota). Agar ek server down bhi ho jaye, tab bhi database chalta rehta hai (Zero Downtime).
- **Alternative Kya Tha?** **Apache Cassandra** ya **MongoDB**
- **Humne Inhe Kyun Reject Kiya?** ScyllaDB actually Cassandra ka hi naya aur upgraded version hai. Cassandra Java me bana tha isliye slow tha; ScyllaDB C++ me hai isliye ye Cassandra se 10-guna fast hai. MongoDB tab use karte jab data bahut zyada nested (JSON) hota, par hume "Raw Speed" chahiye thi, jo ScyllaDB deta hai.

### 3. Elasticsearch (For Notification Worker)

- **Humne Kyun Chuna?** Jab 10,000 employees ko email bhejni ho, toh primary database (ScyllaDB) par loop chalana usko slow kar dega. Elasticsearch ek database kam, aur **"Search Engine"** zyada hai. Ye **'Inverted Index'** ka use karta hai, jisse lakho emails milliseconds me search ho jate hain.
- **Alternative Kya Tha?** **Apache Solr**
- **Humne Ise Kyun Reject Kiya?** Solr bhi achha search engine hai, lekin DevOps aur microservices ki duniya me **ELK Stack (Elasticsearch, Logstash, Kibana)** ka integration sabse best aur famous hai. Logs aur search dono ka kaam Elasticsearch akele sambhal leta hai.

### 4. Redis (For Caching)

- **Humne Kyun Chuna?** Redis ek **"In-Memory"** database hai. Ye data hard-disk par nahi, seedha RAM me save karta hai. Jo data baar-baar read hota hai (jaise company ke common rules, ya user session), use hum Redis me rakh dete hain. RAM se data read karna hard-disk se read karne se hazaron guna fast hota hai.
- **Alternative Kya Tha?** **Memcached**
- **Humne Ise Kyun Reject Kiya?** Memcached bhi RAM me data rakhta hai, lekin agar server restart ho jaye toh uska saara data udd (delete ho) jata hai. Redis ke paas data ko disk par save (persist) karne ka backup feature hota hai. Plus, Redis List, Set aur Hash jaise complex data structures support karta hai jo Memcached nahi karta.

###  Interview ke liye 'Pro-Tip' (Kaise bolein)

Jab interviewer ye sawal pooche, toh jawab is line se shuru karein:

>  Microservices ka pehla rule hai 'Database-per-service'. Isliye humne 'One-size-fits-all' (ek hi database sabke liye) approach nahi lagayi, balki har service ke Use-Case ke hisaab se*
