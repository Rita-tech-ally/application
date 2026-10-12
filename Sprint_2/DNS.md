## These are the AWS Route 53 authoritative name servers for my domain.

I registered my domain `devsecurity.shop` through Hostinger, but I wanted to manage its DNS records using AWS Route 53.

Therefore, I configured these name servers in Hostinger to delegate DNS management to Route 53.

When a DNS resolver looks up my domain, these name servers can provide the DNS records, including the A record that points to my EC2 public IP address.

This allows users to access my frontend application using my custom domain.


## I configured the AWS Route 53 name servers in Hostinger because my domain was registered through Hostinger, but I wanted to manage DNS records using AWS Route 53.

Configuring these name servers delegates DNS authority to Route 53. This allows DNS resolvers to find the DNS records stored in my Route 53 hosted zone, including the A record pointing to my EC2 public IP address.

Without this delegation, the domain might continue using Hostinger's existing DNS name servers, and the Route 53 records would not be used for public DNS resolution.

# DNS Hierarchy: Root DNS, TLD Name Server, and AWS Route 53

## 1. Important Concept

TLD Name Server ke paas us TLD ke under aane wale registered domains ki delegation information hoti hai, lekin har domain ki saari information nahi hoti.

Example: `.shop` TLD ke paas `devsecurity.shop` jaise domains ke authoritative Name Servers ki information hoti hai.

## 2. Kiske Paas Kya Information Hoti Hai?

| Component | Kya information hoti hai? |
|---|---|
| Root Name Server | `.shop` TLD ke Name Servers ki information |
| `.shop` TLD Name Server | `devsecurity.shop` ke authoritative Name Servers kaun se hain |
| AWS Route 53 | Domain ke actual DNS records, jaise A, MX, TXT |
| A Record | Domain ka IPv4 address, jaise `15.252.181.35` |
| AWS EC2 | Tumhari frontend application host hoti hai |

## 3. Tumhare POC Ka Example

```text
Root Name Server
      |
      | .shop TLD ke Name Servers ki information
      v
.shop TLD Name Server
      |
      | devsecurity.shop ke Name Servers ki information
      v
AWS Route 53 Name Servers
      |
      | A record: 15.252.181.35
      v
AWS EC2
      |
      v
NGINX
      |
      v
Frontend Application
```

## 4. Kya Root Ko `.shop` Ke Baare Mein Pehle Se Pata Hota Hai?

Haan! Root DNS system ko `.shop` TLD ke baare mein pehle se pata hota hai. Lekin `devsecurity.shop` ke baare mein information alag se hoti hai.

Ismein 3 baatein yaad rakho:

1. **Root DNS:** Jaanta hai ki `.shop` TLD ke Name Servers kahan milenge.
2. **`.shop` TLD:** Jaanta hai ki `devsecurity.shop` ke authoritative Name Servers kaun se hain.
3. **AWS Route 53:** Jaanta hai ki `devsecurity.shop` ka A record kis IP address ko point karta hai.

TLD ko ye information har waqt automatically pehle se nahi pata hoti. Domain ki registration aur delegation update hone par uski published DNS information update hoti hai.

## 5. Simple Example

Root ko pata hai ki `.shop` ka information desk kahan hai; `.shop` ko pata hai ki tumhare domain ka DNS information desk AWS Route 53 hai.

**Ek line mein yaad rakho:**

- TLD ko pata hota hai ki domain ke DNS records kahan milenge.
- Route 53 ko pata hota hai ki domain kis IP address par point karta hai.

##  NS (Name Server) records DNS resolution mein help karte hain. Inka kaam DNS resolver ko batana hai ki kisi domain ke DNS records kis authoritative Name Server par milenge.

Mere POC mein domain `devsecurity.shop` Hostinger se register kiya gaya hai, lekin DNS management ke liye maine AWS Route 53 use kiya hai. Isliye maine Hostinger mein Route 53 ke four Name Servers configure kiye, taaki domain ki DNS delegation Route 53 ko ho sake.

Jab koi user `devsecurity.shop` access karta hai, toh DNS resolver `.shop` TLD Name Server se domain ke authoritative Name Servers ki information leta hai. Uske baad resolver Route 53 ke Name Server se A record query karta hai, jo domain ko EC2 ke public IP `15.252.181.35` se map karta hai.

Is tarah DNS resolver ko domain ka IP address milta hai aur browser us IP par request bhejkar frontend application access karta hai.
