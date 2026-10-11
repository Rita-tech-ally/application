# Polyglot Architecture in OT-Microservices

Ek hi project me alag-alag languages use karne ki is approach ko **"Polyglot Architecture"** kehte hain. Iska ek hi basic rule hota hai: *"Right tool for the right job"* (Har specific kaam ke liye uski sabse best language chunna).

Agar aap saari APIs ek hi language (jaise sirf Java) me bana dete, toh kuch chote kaamon ke liye wo bahut heavy ho jati, aur kuch processing ke liye slow.

Yahan har specific language ko chunne ka exact technical reason hai:

## 1. Employee API ke liye Golang (Go) Kyun?

- **Kaam:** Ye system ki core API hai jahan sabse zyada traffic aata hai (Read/Write operations).
- **Reason (Speed & Concurrency):** Golang ko Google ne specifically high-performance aur cloud ke liye banaya hai. Isme **"Goroutines"** hote hain. Agar ek saath 10,000 log employee data search karein, toh Java ya Python memory crash kar sakti hain, lekin Go un hazaron requests ko sirf kuch Megabytes (MB) RAM use karke concurrently (ek saath) handle kar leti hai. Ye extremely lightweight aur lightning-fast hai.

## 2. Salary API ke liye Java (Spring Boot) Kyun?

- **Kaam:** Employee ki salary, payroll, aur financial data handle karna.
- **Reason (Security & Enterprise Standards):** Jab baat financial (paise) data ki aati hai, toh industry standard hamesha Java hota hai. Java strictly typed language hai, jisme galti (bugs) hone ke chances Python ya JS se kahin kam hote hain. Dusra bada reason **Spring Boot Framework** hai. Jaise aapne dekha ki Redis caching ke liye sirf `@Cacheable` likhna pada—ye "framework magic" aur enterprise-level security Java ko complex business logic (salary) ke liye best banati hai.

## 3. Attendance API ke liye Python (Flask) Kyun?

- **Kaam:** Daily attendance lagana, time-in/out track karna, aur PostgreSQL ke relational data se deal karna.
- **Reason (Simplicity & Time handling):** Python dates, times, aur complex relational queries (SQL) ko handle karne me bahut flexible aur aasan hai. **Flask** ek "Micro-framework" hai. Attendance API ko Java jaisa bhari-bharkam framework nahi chahiye tha. Flask ne allow kiya ki hum sirf zaroori libraries use karein aur API ko lightweight rakhein.

## 4. Notification Worker ke liye Python Script Kyun?

- **Kaam:** Background me Elasticsearch se data fetch karke bulk (hazaron) emails bhejna.
- **Reason (Scripting & Automation Champion):** Background

# React Frontend, Microservices Communication, and Nginx API Gateway

Maan lijiye company ka admin website par login karke **"Employee Dashboard"** kholta hai, jahan use kisi employee ki Profile (Go), uski is mahine ki Salary (Java), aur aaj ki Attendance (Python) ek hi screen par dekhni hai.

Ye 3 steps me hota hai:

## 1. The API Calls (Frontend Action)

React component (jaise `EmployeeDashboard.js`) load hote hi background me **JavaScript (`fetch` ya `axios`)** ke through 3 alag-alag HTTP requests bhejta hai. Ye requests parallel (ek saath) jati hain:

- `GET http://backend-server:8080/employee/123` (Go ko)
- `GET http://backend-server:8081/salary/123` (Java ko)
- `GET http://backend-server:8082/attendance/123` (Python ko)

*(Kyunki React in sabse directly baat karta hai, isliye ise pata hona chahiye ki kaunsi service kis port par chal rahi hai. Aage chalkar hum isko aasan banane ke liye beech me ek Nginx/API Gateway laga dete hain.)*

## 2. The Aggregation (Data ko jodna)

Go, Java, aur Python—teeno apna-apna kaam karke (database ya Redis se data nikal kar) React ko JSON format me wapas bhej dete hain.

- React un teeno JSON responses ko receive karta hai.
- Ab React ki memory (State/Redux) me 3 alag-alag data packets hain. React code in teeno ko mila kar ek single "State Object" banata hai. Is process ko **Data Aggregation** kehte hain.

## 3. The Rendering (User ko dikhana)

Ab React is mixed data ko apne HTML/CSS structure me fit karke screen par paint (render) kar deta hai.

- User ko lagta hai ki usne ek button dabaya aur poora page ek jagah se aa gaya. Use bilkul pata nahi chalta ki parde ke peeche 3 alag languages aur 2 alag databases ne milkar ye page banaya hai.

### Interview Summary

"Frontend (React) sabhi independent backend services ke sath HTTP/REST calls ke through connect karta hai. Jab koi page load hota hai, toh React parallel API calls karta hai, saari services se JSON data receive karta hai, aur us data ko aggregate (stitch) karke user ko ek unified UI (Single Page Application) dikhata hai."

---

## React ko Backend Services ki Information Kaise Milti Hai?

Frontend (React) khud se kuch nahi janta. Wo bilkul "dumb" hota hai jab tak use explicitly bataya na jaye. React ko backend services se baat karne ke liye do cheezon ki sabse zyada zaroorat hoti hai: **Addresses (Ports/URLs)** aur **Menu Card (API Contract)**.

Is poore process ko hum 3 steps me samajhte hain:

### 1. The Addresses (Environment Variables)

React ko sabse pehle ye batana padta hai ki kaunsi service kahan chal rahi hai. React app ke andar ek `.env` (Environment) file banayi jati hai. Isme saare IP addresses aur ports pehle se save hote hain. Aapke code me ye kuch aisa dikhta hai:

```env
REACT_APP_EMPLOYEE_API_URL=http://backend-server:8080/employee
REACT_APP_SALARY_API_URL=http://backend-server:8081/salary
REACT_APP_ATTENDANCE_API_URL=http://backend-server:8082/attendance
```

Jab React ko Salary ka data chahiye hota hai, toh wo seedha is `.env` file se `8081` wala URL uthata hai.

### 2. The Menu Card (Swagger / OpenAPI)

Ab React ko URL toh mil gaya, par use ye kaise pata ki "data maangna kaise hai" aur "backend JSON kis format me dega"?

Jab aapne Employee API (Go) aur Salary API (Java) me **Swagger UI** configure kiya tha, aur usme `OpenAPIConfig.java` ke andar public servers define kiye the, toh aap asal me frontend developers ke liye ek "Menu Card" (API Contract) bana rahi thi.

- **Swagger Contract:** Ye ek rulebook hoti hai. Ye frontend ko batati hai ki: *"Agar naya employee add karna hai, toh POST request bhejo, aur JSON me exactly `{"name": "...", "email": "..."}` likh kar bhejo. Warna main error de dungi."*
- Frontend developers is Swagger UI documentation ko padhte hain aur bilkul uske rules ke hisaab se apne React code (Axios ya Fetch) me requests likhte hain.

### 3. The Future Upgrade (API Gateway)

Abhi aap Docker Compose me direct 8080, 8081 ports use kar rahi hain. Lekin badi IT companies me frontend kabhi direct ports se baat nahi karta.

- Beech me ek **API Gateway** (jaise Nginx ya Traefik) laga diya jata hai.
- React sirf ek single URL jaanta hai (e.g., `api.mygurukulam.com`).
- React Gateway ko bolta hai `[api.mygurukulam.com/salary/123](https://api.mygurukulam.com/salary/123)`.
- Gateway ko pata hota hai ki "salary" word ka matlab port `8081` hai, toh wo chup-chap us request ko Java service ke paas bhej deta hai. Isse frontend ka code bahut clean aur secure ho jata hai (aur CORS ke issues bhi khatam ho jate hain).

### Summary

React ko ports ki jankari `.env` files se milti hai, aur requests/responses ka JSON format us Swagger API documentation se pata chalta hai jo backend engineers (jaise aap) likhte hain.

---

## Nginx as an API Gateway

Nginx ko API Gateway ki tarah use karna microservices ka sabse best aur standard practice hai. Is setup me Nginx ek "Smart Receptionist" (ya Reverse Proxy) ki tarah kaam karta hai.

Aaiye dekhte hain ye kaise kaam karta hai aur iski configuration (code) kaisi dikhti hai:

### The Problem (Bina Nginx ke)

Abhi aapke React frontend ko 3 alag-alag addresses yaad rakhne padte hain:

- Go: `http://localhost:8080/employee`
- Java: `http://localhost:8081/salary`
- Python: `http://localhost:8082/attendance`

Agar kal ko aap Java ka port `8081` se badal kar `9000` kar dein, toh aapko React ka code change karke use dobara build karna padega. Ye bahut problem wali baat hai.

### The Solution (Nginx ke sath)

Hum inn saare ports ke aage ek Nginx server laga dete hain jo **Port 80 (Standard HTTP port)** par chalta hai.

Ab React ko sirf ek hi address yaad rakhna hai: `[http://api.mygurukulam.com](http://api.mygurukulam.com)` (ya `localhost:80`). React sirf path (`/employee`, `/salary`) badal kar request bhejta hai, aur Nginx use chup-chap sahi port par forward kar deta hai.

### Nginx Configuration File (`nginx.conf`) kaise likhi jati hai?

Aap Nginx ko bas ye simple rules likh kar de dete hain:

```nginx
server {
    listen 80;
    server_name api.mygurukulam.com;

    # Agar URL me /employee aaye, toh request Go (8080) ko bhejo
    location /employee {
        proxy_pass http://localhost:8080;
    }

    # Agar URL me /salary aaye, toh request Java (8081) ko bhejo
    location /salary {
        proxy_pass http://localhost:8081;
    }

    # Agar URL me /attendance aaye, toh request Python (8082) ko bhejo
    location /attendance {
        proxy_pass http://localhost:8082;
    }
}
```

### Iske 3 Sabse Bade Fayde (Interview points)

1. **Single Entry Point:** Frontend ka code bahut clean ho jata hai kyunki use sirf ek base URL use karna hota hai.
2. **CORS Issues Khatam:** Kyunki frontend (React) aur backend dono ab ek hi port (Port 80) se baat kar rahe hain, isliye browser ko lagta hai ki saara data ek hi jagah se aa raha hai, aur wo CORS error (jo Java Spring Boot me aapko aaya tha) kabhi nahi deta.
3. **High Security:** Nginx ke aane se aap baaki saare ports (8080, 8081, 8082) ko internet (firewall) se block kar sakte hain. Duniya ko sirf Nginx (Port 80) dikhega, aapke internal APIs bilkul safe aur hidden rahenge.

### Interview ke liye "The Perfect Answer"

**Aapka Jawab:** "Sir/Ma'am, agar Redis fail hota hai, toh meri API crash nahi hogi kyunki humne **Fallback Mechanism** implement kiya hua hai.

Lekin API ke behavior me do main changes aayenge:

1. **Latency Badhegi:** API pehle Redis ke connection timeout (e.g., 1-2 seconds) ka wait karegi, aur fir database ko query karegi.
2. **Database Par Load Aayega:** API saari aane wali requests seedha ScyllaDB ko forward karne lagegi.

Is situation ko handle karne ke liye main Go/Python me `try-catch/error-handling` blocks ka use karungi aur Java Spring Boot me `CacheErrorHandler` configure karungi, taaki Redis fail hone par bhi user ko 200 OK response mile, bas thodi si latency ke sath."

# What Happens If PostgreSQL Goes Down?

Agar PostgreSQL crash ya down ho jata hai, toh aapke system me ek bahut hi interesting situation banegi, jo asal me **Microservices Architecture ki sabse badi jeet (success) hai**.

Isey 4 main points me samajhte hain:

### 1. The Microservices Magic (Fault Isolation)

Agar ye ek purana Monolithic (single) application hota, toh Postgres ke marte hi poori company ka system down ho jata. Lekin aapke OT-Microservices me:

- **Employee API (Go) aur Salary API (Java) 100% normal chalenge.** Kyunki unka database (ScyllaDB) alag hai. Employees apni profile dekh payenge aur salary calculate hoti rahegi. Is concept ko technical bhasha me **"Fault Isolation" (Bimari ko ek jagah rokna)** kehte hain.

### 2. "Write" Requests Fail Hongi (Marking Attendance)

Kyunki PostgreSQL Attendance API (Python Flask) ka main database hai, agar koi employee subah "Punch In" ya "Mark Attendance" button dabayega, toh:

- Flask API Postgres se connect karne ki koshish karegi aur fail ho jayegi (Connection Refused).
- API seedha frontend ko **HTTP 500 (Internal Server Error)** ya 503 (Service Unavailable) return kar degi. Nayi attendance save nahi hogi.

### 3. "Read" Requests (Viewing Attendance) - Redis aayega bachane!

Agar koi employee apni pichle hafte ki attendance dekhne ke liye dashboard kholta hai:

- Agar wo data **Redis (Cache)** me pehle se save hai, toh Flask API chup-chap Redis se data uthayegi aur employee ko dikha degi. User ko pata bhi nahi chalega ki main database down hai!
- Lekin agar data Redis me nahi mila (Cache Miss), tab Flask API Postgres ke paas jayegi, aur tab error aayega.

### 4. Frontend (React) par kya dikhega?

Aapka React frontend bahut gracefully react karega. Dashboard par Employee Profile aur Salary ke boxes normally render honge, lekin Attendance wale box me ek error message aa jayega: *"Attendance service is temporarily down. Please try again later."*

## Interview ke liye "The Architect Answer"

"Sir/Ma'am, Postgres down hone par sirf Attendance API ka 'Write' operation down hoga. Reads tab tak kaam karenge jab tak data Redis me zinda hai. Sabse bada fayda ye hai ki Microservices ki wajah se humara baaki system (Employee aur Salary) bilkul unaffected rahega (Fault Isolation). Halanki, write requests fail hone se bachane ke liye humein aage chal kar API aur Database ke beech ek Message Queue (jaise Kafka ya RabbitMQ) lagani padegi."

## Interview Summary: Python as an Interpreted Language

"Python ek interpreted language hai kyunki iska execution line-by-line hota hai. Code pehle platform-independent bytecode me convert hota hai, aur fir Python Virtual Machine (PVM) us bytecode ko runtime par machine code me interpret karke execute karti hai, jiske karan ye compiled languages (jaise Go) se slow hoti hai par developer-friendly hoti hai."

# Python Execution, Compiler vs Interpreter, and Database Drivers

## Interview ke liye "The Architect Answer"

"Sir/Ma'am, technically Python purely interpreted nahi hai, ye **Compiled aur Interpreted ka mixture** hai. Jab hum script run karte hain, toh Python pehle source code ko automatically ek intermediate 'Bytecode' (`.pyc`) me compile karta hai, aur uske baad Python Virtual Machine (PVM) us bytecode ko runtime par line-by-line interpret karti hai."

## Compiler vs Interpreter

| **Feature** | **Compiler** | **Interpreter** |
|---|---|---|
| **Execution Style** | Poore code ko ek hi baar (bulk) me scan karta hai aur translate karta hai. | Code ko line-by-line padhta hai aur turant execute karta hai. |
| **Speed** | Runtime par bahut fast hota hai kyunki translation pehle hi ho chuka hota hai. | Runtime par slow hota hai kyunki execution ke sath-sath translation (translation-on-the-fly) chal raha hota hai. |
| **Error Handling (Debugging)** | Saare errors ek sath end me dikhata hai. Agar 10 errors hain, toh poora scan hone ke baad hi pata chalega. | Jaise hi kisi line me error aata hai, ye wahi ruk jata hai. Isliye line-by-line debugging aasan hoti hai. |
| **Output File** | Ek binary executable file (`.exe` ya machine code) generate karta hai jise aap bina compiler ke bhi run kar sakte hain. | Koi standalone executable file generate nahi karta. Ise run karne ke liye hamesha interpreter ki zaroorat padti hai. |
| **Memory (RAM)** | Memory zyada leta hai kyunki poore program ka object/machine code RAM me load hota hai. | Memory kam leta hai kyunki ek baar me sirf ek line ka code process hota hai. |
| **Best Examples** | C, C++, Golang (Employee API), Rust. | JavaScript, Ruby, PHP. |



# Attendance API Deployment on AWS EC2 Using Poetry, Liquibase, Gunicorn, and Systemd

## 1. Short Answer (30-Second Pitch)

*"Maine AWS EC2 Ubuntu server par Python, PostgreSQL aur Redis setup kiya. Uske baad Liquibase se database tables migrate kiye, aur aakhir me Gunicorn aur Systemd ka use karke API ko port 8080 par as a background service UP kar diya."*

## 2. Detailed Explanation (Interview Style)

Agar wo bolein ki *"Mujhe step-by-step samjhao"*, toh aap aise batayengi:

### Step 1: Infrastructure & Dependencies (Zameen taiyar karna)

"Sabse pehle maine ek fresh AWS EC2 Ubuntu instance liya aur uski Security Group me port `8080` aur `22` open kiye. Phir maine server par Python 3.11, Java 17 (Liquibase ke liye), PostgreSQL aur Redis install kiye."

### Step 2: Code & Environment Setup (Code laana)

"Uske baad maine GitHub se Attendance API ka code clone kiya. Python packages me aapas me conflict na ho, isliye maine **Poetry** ka use karke ek isolated virtual environment banaya aur saari dependencies install ki."

### Step 3: Database & Migrations (Data ki jagah banana)

"Phir maine Postgres me `attendance_user` aur `attendance_db` create kiya aur Redis ko start kiya. Database me manually tables banane ke bajaye, maine **Liquibase** (`liquibase update` command) ka use kiya, jisne automatically saare schema aur tables create kar diye."

### Step 4: The 'UP' Process (API ko live karna)

"API ko simple terminal me chalane ki bajaye (kyunki terminal band hote hi API band ho jati), maine **Gunicorn** server ka use kiya. Maine Ubuntu me ek **Systemd Service** (`attendance-api.service`) banayi."

### Step 5: Final Validation (Check karna)

"Aakhir me, maine `systemctl start attendance-api` command se us service ko UP kiya aur enable kar diya. Iska sabse bada fayda ye hai ki agar server restart bhi ho jaye, toh meri API automatically UP ho jayegi. Phir maine browser me Health check URL aur Swagger UI khol kar verify kar liya ki sab perfect chal raha hai."

---

## 3. Why Poetry Instead of pip and requirements.txt?

Python me packages (libraries) install karne ke liye purana aur standard tareeka **`pip`** aur **`requirements.txt`** hai. Lekin apne Attendance API project me aapne **Poetry** ka use kiya hai kyunki ye aajkal ki industry ka sabse modern aur "Smart" dependency manager hai.

Ise use karne ke 3 sabse bade DevOps aur Engineering reasons hain:

### 3.1. "It works on my machine" wali problem khatam (`poetry.lock`)

Normal `pip` ke sath sabse badi problem ye hoti hai ki developer ke laptop par code chal jata hai, par AWS server par crash ho jata hai kyunki kisi library ka version alag install ho gaya.

Poetry ek **`poetry.lock`** file banata hai. Ye file is baat ki 100% guarantee deti hai ki jo exact version aapke laptop par install hua tha, bit-by-bit wahi exact version EC2 server par bhi install hoga.

### 3.2. Dependency Resolution (Smart Installation)

Maan lijiye aapki API me do packages hain: **Flask** aur **SQLAlchemy**. Agar dono ko kisi teesre package ka alag-alag version chahiye, toh `pip` pehle wale ko install kar dega aur baad me error dega (Dependency Hell).

Poetry bahut smart hai. Install karne se pehle ye pura math calculate karta hai aur dekhta hai ki kaunsa version dono ke sath perfectly match karega, aur tabhi install karta hai.

### 3.3. System Python ko bachana (The Ubuntu Problem)

Aapke POC document me ek line thi: *"without changing system default Python"*.

Ubuntu operating system apne internal kaamo ke liye Python par depend karta hai. Agar aap `pip install` se OS wale Python me ched-chad karte hain, toh poora EC2 server crash ho sakta hai.

Poetry automatic tarike se aapke project ka ek alag **Isolated Virtual Environment** bana deta hai (`poetry env use python3.11`). Isse aapka API ka Python OS ke Python se bilkul alag aur safe rehta hai.

## 4. Interview ke liye "The Architect Answer"

"Sir/Ma'am, hum chahte toh traditional `pip` use kar sakte the, par humne **Poetry** isliye chuna kyunki ye dependency management ko predictable banata hai. Iski `poetry.lock` file ensure karti hai ki Development aur Production environment bilkul identical hon. Sath hi, ye automatic virtual environments manage karta hai jisse humara system Python corrupt hone se bach gaya aur CI/CD pipelines me version conflicts nahi aaye."


# Understanding `pyproject.toml` in Poetry

**`pyproject.toml`** hi Poetry ka "Dil aur Dimaag" (Heart and Brain) hai. Jo kaam pehle `requirements.txt` karti thi, ab wo saara kaam ye single file karti hai, par kahin zyada smart tareeke se.

Aaiye dekhte hain ki ye purani `requirements.txt` se kaise alag aur behtar hai:

## 1. Ek File, Saara Kaam (All-in-One)

Purane Python projects me bahut saari files hoti thi—dependencies ke liye `requirements.txt`, package banane ke liye `setup.py`, aur configuration ke liye `setup.cfg`. Poetry in sabko hata kar sirf ek **`pyproject.toml`** file banata hai. Ye aapke poore project ka single source of truth hai.

## 2. Project Metadata

`requirements.txt` ek "dumb" file hoti hai; usme sirf packages ke naam likhe hote hain (jaise `Flask==2.0`). Lekin `pyproject.toml` me aapke project ki poori kundali (metadata) hoti hai:

- Project ka naam kya hai? (Attendance API)
- Version kya hai? (1.0.0)
- Author kaun hai? (Aapka naam)
- Python ka exact kaunsa version chahiye? (Python 3.11)

## 3. "Dev" vs "Production" Packages (Sabse Bada Fayda)

`requirements.txt` me aapke saare packages mix ho jate hain. Lekin `pyproject.toml` me aap packages ko 2 alag hisson me baant sakti hain:

- **Main Dependencies:** Wo tools jo server par chalne ke liye chahiye (jaise `Flask`, `gunicorn`, `scylla-driver`).
- **Dev Dependencies:** Wo tools jo sirf aapko apne laptop par testing ya code formatting ke liye chahiye (jaise `pytest`, `black`). Jab aap code ko AWS (Production) par bhejti hain, toh Poetry sirf Main Dependencies install karta hai, jisse server ka size chota aur fast rehta hai.

## 4. Smart Version Ranges

`requirements.txt` me agar aapne likh diya `requests==2.25.0`, toh wo wahi fixed version install karega, chahe usme security bug hi kyun na ho. `pyproject.toml` me aap range de sakti hain: `requests = "^2.25.0"`. Iska matlab Poetry ko permission mil jati hai ki *"2.25.x ka koi bhi latest aur safe version install kar do, par 3.0.0 par mat jana kyunki code break ho sakta hai."*

## 5. Interview ke liye "The Architect Answer"

"Sir/Ma'am, `pyproject.toml` sirf ek text file nahi hai, ye Python ka naya standard (PEP-518) hai. Humne `requirements.txt` ki jagah iska use isliye kiya kyunki ye sirf dependencies ko nahi, balki project ke metadata, build system, aur development-vs-production tools ko ek hi jagah centrally aur neatly manage karti hai."


# Polyglot Microservices Architecture and Nginx as an API Gateway

Nginx sirf ek **"Traffic Police" (ya Receptionist)** hai, aur aapke alag-alag frameworks (Go, Java, Python) us office ke **"Specialist Workers"** hain.

Hum alag-alag frameworks isliye use nahi kar rahe kyunki hum Nginx use kar rahe hain. Asal me hum **Polyglot Architecture (Right tool for the right job)** use kar rahe hain, aur Nginx us complexity ko frontend (React) se chupane ka kaam kar raha hai.

Ise 2 main reasons se samajhte hain:

## 1. Har Framework ki Apni "Superpower" hai (Polyglot Microservices)

Agar hum chahte toh teeno APIs sirf Java ya sirf Python me bana sakte the. Lekin ek bade enterprise project me har API ka kaam alag hota hai:

- **Employee API (Go):** Yahan humein raw speed aur high concurrency chahiye thi taaki hazaron log ek sath apni profile tezi se dekh sakein. Go ke **Goroutines** iske liye best hain.
- **Salary API (Java Spring Boot):** Salary ka data financial aur complex hota hai. Java bahut strict (strongly typed) language hai aur Spring Boot ka ecosystem security aur financial calculations ke liye industry standard (bulletproof) maana jata hai.
- **Attendance API (Python Flask):** Attendance ka data simple hota hai. Python me development bahut fast hoti hai. Kal ko agar HR ko attendance data par Data Analytics ya AI (jaise pattern recognition ki kaun kab late aata hai) lagana ho, toh Python wahan sabse best tool hai.

## 2. Nginx ka Asli Kaam (API Gateway Pattern)

Kyunki aapne 3 alag bhashayein aur 3 alag ports (8080, 8081, 8082) use kiye hain, React frontend ke liye ye bahut badi sir-dard (headache) ban jata. React ko yaad rakhna padta ki Employee ke liye kahan jana hai aur Salary ke liye kahan.

Nginx as an **API Gateway** is problem ko solve karta hai:

- React frontend sirf Nginx (`localhost:80`) ko request bhejta hai.
- Frontend ko pata hi nahi hota ki parde ke peeche Go chal raha hai ya Java.
- Nginx URL dekhta hai: Agar request `/api/employee` par aayi, toh wo chup-chap use Go (8080) par bhej deta hai. Agar `/api/salary` aayi, toh Java (8081) par bhej deta hai.

## 3. Interview ke liye "The Architect Answer"

"Sir/Ma'am, humne **Polyglot Microservices Architecture** apnaya hai, jiska rule hai *'Right Tool for the Right Job'*. Go speed ke liye, Java strict financial transactions ke liye, aur Python fast development ke liye. Kyunki alag-alag tech-stack hone ki wajah se ports aur routing complex ho jati hai, humne **Nginx ko as an API Gateway (Reverse Proxy)** use kiya hai. N
