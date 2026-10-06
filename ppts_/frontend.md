# . Frontend Web (README / Presentation Script)

### 1. Introduction (Frontend kya hai)

Frontend Web OT-Microservices stack ka mukhya **User Interface (UI)** hai. Ye ek **ReactJS** base web application hai jo users aur admins ko data dekhne aur manage karne ke liye dashboard provide karta hai. Iska backend apna koi nahi hota, ye seedha Employee API, Attendance API aur Salary API se baat karta hai.

### 2. Tech Stack Explanation (Point-by-Point)

- **ReactJS:** User interface create karne ke liye component-based Javascript library. Isse single-page application (SPA) banti hai jo bina page reload kiye fast chalati hai.
- **Tabler-React:** Ek open-source admin dashboard UI kit hai jo built-in beautiful UI components (Cards, Tables, Grids, Badges) deti hai, taaki UI jaldi ban sake.
- **C3.js (react-c3js):** Dashboard pe dikhne wale interactive donut aur bar charts (jaise Role distribution, Location distribution) generate karne ke liye ye charting library use hui hai.
- **Reactstrap / Formik:** Forms (Add Employee, Add Attendance) banane aur validate karne ke liye use hota hai.

### Step-by-Step Explanation

1. **User dashboard open karta hai (Browser pe)**
2. React app load hota hai aur `HomePage.react.js` render karta hai.
3. Page load hote hi React ke **Lifecycle methods / Hooks** (`useEffect` aur `fetch()`) background me calls karte hain:
   - `GET /employee/search/all`
   - `GET /employee/search/status`
4. **Backend se data milne par:** React ka *State* update hota hai (`this.setState` ya `handleStats`).
5. State update hote hi C3.js ke charts (Role, Location) aur UI ki Tables (EmployeeList, AttendanceList) automatically **re-render** ho jati hain aur data screen par show ho jata hai.
6. **Form Submit karne pe:** Jab admin naya employee banata hai, to Formik `POST /employee/create` aur `POST /notification/send` par JSON payload bhej deta hai taaki data backend pe save ho jaye aur us user ko welcome notification chala jaye.

### Kyun Ye Approach Use Ki Gayi

- **Decoupled Architecture:** Frontend ka code backend code se completely alag hai (`package.json` vs `go.mod`/`pom.xml`). React API calls ke through communication karta hai. Iska faayda ye hai ki backend me kal ko Go ki jagah Node.js bhi lag jaye, to Frontend ka UI waise hi chalta rahega.

## Frontend aur Backend Microservices ka Connection

Frontend aur baaki backend microservices (Employee, Salary, Attendance, Notification) ke beech ka connection **REST APIs (HTTP Requests)** ke through hota hai.

Kyunki ye ek decoupled (alag-alag) architecture hai, Frontend seedha database se baat nahi karta. Iski jagah, ReactJS components Javascript ke in-built `fetch()` function ka use karke backend services ko HTTP `GET` aur `POST` requests bhejte hain.

Yahan detail me samajhiye ki ye connection code aur architecture level par kaise kaam kar raha hai:

### 1. Code Level Connection (The `fetch` API)

Agar aap frontend ka code dekhein (jaise `EmployeeList.js`, `ListSalary.js`, ya `AttendanceList.js`), to wahan backend ka data laane ke liye `fetch()` API ka use hua hai.

**Example: Salary Data Fetch Karna (`ListSalary.js`)**

```javascript
loadData() {
    // Frontend Salary API ke endpoint pe GET request bhej raha hai
    fetch('/salary/search/all')
        .then(response => response.json())
        .then(data => {
            this.setState({data: data }) // Data milne par UI update
    })
}
```

**Example: Multiple Services ko Ek Saath Call Karna (`EmployeeForm.js`)**

Jab admin ek naya employee add karta hai, to frontend ek hi submit button par **do alag-alag microservices** ko parallel me call karta hai:

```javascript
handleSubmit(values) {
    // 1. Employee API ko data bhejta hai (ScyllaDB me save karne ke liye)
    fetch('/employee/create', { method: 'POST', body: JSON.stringify(values) })
    
    // 2. Notification Worker ko trigger karta hai (Welcome email bhejne ke liye)
    fetch('/notification/send', { method: 'POST', body: JSON.stringify(values) })
}
```

### 2. Architecture Level Connection (API Gateway / Reverse Proxy)

Kyunki har microservice alag port aur alag language me chal rahi hai (jaise Employee API Go me 8080 par, Salary API Java me kisi aur port par), to frontend directly alag-alag ports par request nahi bhejta (warna CORS errors aayenge).

Is problem ko solve karne ke liye aapke architecture me do tarike use hue hain:

- **Local Development Me (`package.json` proxy):** Aapke `package.json` me `"proxy": "http://localhost:3000"` likha hai. Iska matlab hai jab frontend `/employee/create` par request bhejta hai, to React ka development server automatically is request ko port `3000` par forward kar deta hai.
- **Production Environment Me (Reverse Proxy / API Gateway):** Production me, ek NGINX server ya API Gateway (jaise Kubernetes Ingress) frontend aur backends ke beech me baithta hai aur URL ke hisaab se traffic route karta hai:
  - Agar request `/employee/*` aati hai → Go API (Employee) ko bhej do.
  - Agar request `/salary/*` aati hai → Java API (Salary) ko bhej do.
 
