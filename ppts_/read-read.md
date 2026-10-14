# Database Selection in OT-Microservices (The 100% Correct Version)

**Interviewer ke samne Pitch (Shuruwat):**

> Microservices ka pehla rule hai 'Database-per-service'. Isliye humne ek hi database sabke liye use nahi kiya, balki har service ki speed aur data-structure ke hisaab se uski best technology chuni hai..."*

### 1. ScyllaDB (For Employee & Salary APIs)

- **Humne Kyun Chuna?** Employee aur Salary dono hamari highly critical services hain jahan data ka volume bahut zyada hai. Salary API ko month-end par bulk me hazaron calculations aur entries milliseconds me karni hoti hain. ScyllaDB C++ me bana hai aur "Thread-per-core" architecture use karta hai, jo extreme read/write speed deta hai. Sath hi, iska **Masterless Architecture** ensure karta hai ki system me Zero-Downtime rahe.
- **Alternative Kya Tha?** Apache Cassandra.
- **Humne Ise Kyun Reject Kiya?** ScyllaDB actually Cassandra ka hi naya version hai. Cassandra Java me bana tha isliye usme 'Garbage Collection' delays aate the. ScyllaDB C++ me hai isliye ye Cassandra se 10-guna fast hai.

### 2. Redis (For Salary API Caching)

- **Humne Kyun Chuna?** Salary API ko calculations ke waqt kuch data baar-baar chahiye hota hai (jaise tax rules, PF percentages, ya common company policies). Agar iske liye baar-baar ScyllaDB par query lagayenge toh system slow hoga. Isliye humne beech me **Redis (In-Memory Database)** lagaya hai. Ye data ko direct RAM me rakhta hai, jisse Salary API ka response time hazaron guna fast ho jata hai.
- **Alternative Kya Tha?** Memcached.
- **Humne Ise Kyun Reject Kiya?** Memcached bhi RAM me data rakhta hai, lekin server restart hone par usme saara data udd jata hai. Redis ke paas data ko disk par save (persist) karne ka feature hai, plus ye complex data structures (Lists, Hashes) support karta hai.

### 3. PostgreSQL (Only For Attendance API)

- **Humne Kyun Chuna?** Employee aur Salary ka data flat/NoSQL me fit ho jata hai, lekin Attendance ka data purely relational hota hai. Ek employee ke daily punch-in, punch-out, aur monthly leave records me aapas me relations hote hain. Is data par complex JOINs, filters, aur SQL queries chalani padti hain, jiske liye NoSQL achha nahi hai. Isliye sirf is API ke liye humne 100% ACID compliant SQL database (PostgreSQL) use kiya.
- **Alternative Kya Tha?** MySQL ya Oracle DB.
- **Humne Inhe Kyun Reject Kiya?** Oracle DB enterprise level par bahut expensive (mehnga) aur heavy hai, jabki Postgres completely free aur open-source hai. Rahi baat MySQL ki, toh wo ek achha option tha, lekin Attendance system me **Date-Time operations aur complex queries** (jaise daily hours calculate karna ya weekly leave reports nikalna) bahut zyada hoti hain. In advanced SQL features aur strict data integrity ke mamle me PostgreSQL ka engine MySQL se zyada powerful aur robust hai.

### 4. Elasticsearch (For Notification Worker)

- **Humne Kyun Chuna?** Jab thousands of employees ko email bhejni ho, toh primary database (ScyllaDB) se mails nikalna usko slow kar dega. Elasticsearch ek **"Search Engine"** hai jo 'Inverted Index' use karta hai. Isse lakho emails milliseconds me search ho jate hain bina hamare main DB ko disturb kiye.
- **Alternative Kya Tha?** Apache Solr.
- **Humne Ise Kyun Reject Kiya?** Solr bhi achha hai, par DevOps me **ELK Stack (Elasticsearch, Logstash, Kibana)** ka integration sabse best hai. System logs aur search dono ek hi tool (ES) sambhal leta hai.

