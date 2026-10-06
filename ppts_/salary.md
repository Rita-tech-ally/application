1. Salary API (README / Presentation Script)
# 1. Introduction (Salary API kya hai)
Salary API ek backend microservice hai jo company ke employees ki salary, payroll records aur compensation transactions manage karti hai. Ye Java 17 me Spring Boot framework use karke bani hai. Ye API completely platform-independent hai aur REST endpoints ke through data serve karti hai. OT-Microservices stack me ye independently deploy hoti hai aur apna data isolation maintain karti hai.

# 2. Tech Stack Explanation (Point-by-Point)
Java + Spring Boot: Application Java me bani hai aur Spring Boot framework use karti hai jo dependency injection, routing, aur web server (Tomcat) jaisi cheezein aasaani se handle karta hai. Enterprise applications ke liye ye ek highly stable aur secure approach hai.

## Primary database — ScyllaDB: Employee ki salary ka data ScyllaDB me store hota hai. Ye high-volume data ko low-latency ke saath read/write karne ke liye best hai.

Cache — Redis: Search performance badhane ke liye Spring Cache ke saath Redis integrate kiya gaya hai. Jo data baar-baar read hota hai (jaise specific ID ki salary), wo Redis me cache ho jata hai.

Swagger API documentation (springdoc): /salary-documentation endpoint pe API ka UI open hota hai jaha frontend developers ya QA team API endpoints ko test aur understand kar sakti hai.

Health & Metrics: Spring Boot Actuator (/actuator/health, /actuator/prometheus) application ki health, ScyllaDB/Redis ki connectivity aur performance metrics monitor karne me help karta hai.

Migration — golang-migrate: Database tables ka structure create karne ke liye golang-migrate tool ka use kiya gaya hai.

# Step-by-Step Explanation
Client request bhejta hai — jaise GET /api/v1/salary/search?id=EMP001.

Spring Controller request receive karta hai (SpringDataController.java) aur use service layer (SpringDataSalaryService.java) ke paas bhejta hai.

Service layer logic chalata hai: Sabse pehle wo @Cacheable("salary-search-id") annotation ke through check karta hai ki data Redis me hai ya nahi.

Cache check:

Cache HIT: Data Redis se milta hai aur fast response chala jata hai.

Cache MISS: Request Cassandra/ScyllaDB Repository ke paas jati hai.

Database Query: Repository interface ScyllaDB me query chalata hai (SELECT * FROM employee_salary WHERE id = ?).

Cache Update & Response: Naya data Redis me save ho jata hai aur client ko JSON format me response bhej diya jata hai.

Kyun Ye Approach Use Ki Gayi
Spring Boot enterprise-level security aur structure provide karta hai.

ScyllaDB + Redis ka combo heavy load pe bhi application ko fast rakhta hai aur database bottleneck se bachata hai.

# Run Migrations — What Happens
Pre-requisite: employee_db keyspace ScyllaDB me bana hona chahiye.

make run-migrations chalate ho

migrate tool migration.json se ScyllaDB ka connection string leta hai.

migration/ folder se .sql files (CREATE TABLE employee_salary...) leta hai.

Check karta hai purani migrations aur pending migrations ko apply kar deta hai.

Result: employee_salary table ScyllaDB me ready ho jati hai.


## Salary API ke 5 primary endpoints hain. Inme se 3 endpoints data operations (CRUD) ke liye hain aur 2 application monitoring ke liye hain.

POST /api/v1/salary/create/record: Naya salary record database me create karne ke liye.


GET /api/v1/salary/search: Employee ID (query parameter) ke basis par kisi specific employee ki salary retrieve karne ke liye.


GET /api/v1/salary/search/all: System ke sabhi employees ke salary records ek saath fetch karne ke liye.


GET /actuator/health: Application aur uski dependencies (jaise ScyllaDB, Redis) ka health status check karne ke liye.


GET /actuator/prometheus: Prometheus monitoring system ke liye performance metrics expose karne ke liye.

# Spring Boot aur Tomcat ka Relation (Limitation ka Confusion)

Ye ek misconception hai ki Spring Boot ki kisi "limitation" ki wajah se Tomcat use hota hai. Asal mein, Spring Boot aur Tomcat ek doosre ke competitors nahi, balki partners hain.


Spring Boot koi web server nahi hai; wo sirf ek framework hai jo apka code structure karta hai. Kisi bhi Java web application ko network par HTTP requests (GET, POST) handle karne ke liye ek Web Server (Servlet Container) ki zarurat padti hi hai.


Spring Boot ka sabse bada fayda hi ye hai ki wo Tomcat ko "embedded" (in-built) form me apko deta hai. Pehle ke time me developers ko alag se Tomcat server install karke usme apna code deploy karna padta tha. Ab Spring Boot me aap code run karte hain, aur uska in-built Tomcat khud start ho kar aapki API ko (jaise port 8080 par) host kar deta hai.


## Dono ki Definitions

Spring Boot: Ye ek modern Java framework hai jo complex enterprise applications ka setup aur configuration automatically handle kar leta hai. Iska main kaam developers ko lamba setup aur boilerplate code likhne se bachana hai, taaki wo seedha "production-ready" REST APIs aur microservices bana sakein.

Apache Tomcat: Ye ek open-source Web Server aur Servlet Container hai. Iska kaam client (browser, mobile app, ya frontend) se aane wali HTTP requests ko receive karna, unhe Spring Boot application tak pahunchana, aur application ke generated response (jaise JSON data) ko wapas client tak bhejna hai.


Spring Boot is a powerful Java framework, and its relationship with Tomcat is a key reason why it's so popular for building web applications and microservices like the Salary API.

## Spring Boot and Tomcat: How They Work Together
It is a common misconception that Tomcat is used because of a "limitation" in Spring Boot. In reality, Spring Boot and Tomcat are partners, not competitors.

Spring Boot itself is not a web server; it is a framework that structures your code, manages dependencies, and handles configuration. However, any Java web application needs a Web Server (Servlet Container) to listen to network ports and handle incoming HTTP requests (like GET or POST).

The major innovation of Spring Boot is that it provides an embedded web server. In the past, developers had to manually install a standalone web server (like Tomcat), package their application into a WAR (Web Application Archive) file, and deploy it to that server. Spring Boot revolutionized this by embedding the server directly into the application. When you run a Spring Boot app, the embedded server starts automatically and hosts the application.

Apache Tomcat is the default embedded server in Spring Boot. It is used because it is a highly reliable, widely adopted, and robust servlet container. It manages TCP connections, handles network I/O, and uses a thread pool to process incoming HTTP requests efficiently.
