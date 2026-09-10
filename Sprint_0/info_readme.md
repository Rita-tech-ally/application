#### apt ek package manager hai jo internet par मौजूद official ya configured repositories (repos) se software download aur install karta hai.

Yahan iska poora process aur source kaam karne ka tarika hai:

### 1. Packages kahan se aate hain? (Repositories)
Aapke system ke andar ek file hoti hai (aamtaur par /etc/apt/sources.list ya /etc/apt/sources.list.d/ ke andar), jisme alag-alag servers ke web links (URLs) likhe hote hain. Inhi servers ko Repositories ya Repo kehte hain.

Ye servers Ubuntu ya Debian ke official cloud servers hote hain, jahan hazaron software (jaise git, nginx, jq) compressed files (.deb format me) ke roop me rakhe hote hain.

### 2. apt update ka kya kaam hai?
Jab aap sudo apt update chalate hain, toh apt inhi official servers par jaakar ek list download karta hai jisme sabhi softwares ke latest versions ki jankari hoti hai. Ye list aapke system ke local cache me save ho jati hai taaki apt ko pata chal sake ki kaunsa version available hai.

## 3. apt install kaise kaam karta hai?
Jab aap sudo apt install <package_name> likhte hain, toh ye steps hote hain:

**Dependency Check:** Apt check karta hai ki us software ko chalane ke liye kisi aur support file ya library ki zarurat toh nahi hai (jaise agar koi tool Python par dependent hai, toh apt usko bhi check karega).

**Download:** Apt us official server se us software ki .deb file ko download karta hai.

**Installation:** Download hone ke baad apt us file ko uncompress karke system ki sahi jagah par set kar deta hai (jaise binaries ko /usr/bin ya /usr/local/bin me daal deta hai).
 isa acha se  structure me do taki readme .md me dall sku
