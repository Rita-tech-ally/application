| Step | Activity             | What Happens                                                                                                        | Why It Is Important                                                                                |
| ---- | -------------------- | ------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| 1    | **Ansible Syntax Check** | `ansible-playbook --syntax-check` command checks the playbook for YAML syntax, indentation, and structural errors.  | Detects basic syntax errors early and prevents the pipeline from moving forward with invalid code. |
| 2    | **Ansible Linting**      | `ansible-lint` scans the Ansible code against coding standards and best practices.(jaise variables ko sahi se naam dena, ya purane/deprecated modules use na karna). Yeh code ko clean rakhta hai.                                  | Helps maintain clean, consistent, and high-quality Ansible code.                                   |
| 3    | **Dry-Run / Check Mode** | The playbook runs with `ansible-playbook --check` without making actual changes to the target servers.              | Shows what changes would be made and helps identify potential issues safely.                       |
| 4    | **Required Tests**       | The Ansible role/playbook is tested in a temporary environment such as Docker containers using tools like Molecule. | Verifies that the automation works correctly in a real test environment.                           |
| 5    | **CI Pipeline Result**   | The CI tool collects the results and logs from all checks and generates a final pipeline status.                    | Helps developers identify which checks passed or failed and troubleshoot errors.                   |
| 6    | **Pull Request Updated** | The CI result is reported back to GitHub/GitLab, showing the PR status as passed or failed.                         | Allows reviewers to quickly verify whether the code has passed the required checks.                |
| 7    | **Code Merging**         | The code can be merged into the main branch only when all required CI checks pass.                                  | Prevents untested or faulty code from being merged into the main branch.    |


----

# Test	Runs Molecule and functional tests to verify the Ansible Role.


| Test                                 | Kya check karta hai?                                                 | Example                                  |
| ------------------------------------ | -------------------------------------------------------------------- | ---------------------------------------- |
| **1. Role/Playbook Functional Test** | Role expected configuration apply kar raha hai ya nahi               | Nginx install hua ya nahi                |
| **2. Service Test**                  | Required service properly running hai ya nahi                        | `systemctl is-active nginx`              |
| **3. Configuration Test**            | Configuration file correct hai ya nahi                               | `/etc/nginx/nginx.conf` exists and valid |
| **4. File/Directory Test**           | Required files/directories create hue ya nahi                        | `/var/www/html` exists                   |
| **5. Package Test**                  | Required packages installed hain ya nahi                             | `nginx`, `curl` installed                |
| **6. Permission/Ownership Test**     | File permissions and ownership correct hain ya nahi                  | `root:root`, `0644`                      |
| **7. Idempotency Test**              | Role ko dobara run karne par unnecessary changes nahi hone chahiye   | Second run → `changed=0`                 |
| **8. Connectivity Test**             | Required service/port accessible hai ya nahi                         | `curl http://localhost`                  |
| **9. Molecule Tests**                | Role ko temporary test environment me provision karke test karta hai | Molecule + Docker                        |
| **10. Custom/Automated Tests**       | Project-specific requirements verify karta hai                       | Application config, ports, users etc.    |

---

## Molecule test Ansible Role ki automated testing framework hai.

Simple words mein:

##### Molecule ka use Ansible Role ko ek temporary/test environment mein run karke verify karne ke liye hota hai ki role properly kaam kar rahi hai ya nahi.

Example

### Maan lo tumhari Ansible Role ka naam nginx hai.

Molecule ye process follow kar sakta hai:

Molecule Test
     ↓
Test Container/VM Create
     ↓
Ansible Role Run
     ↓
Nginx Install & Configure
     ↓
Verify Tests
     ↓
Idempotency Check
     ↓
Test Environment Destroy
Molecule kya-kya check kar sakta hai?

**1** Role execute ho rahi hai

nginx package installed

**2.** Service running hai

nginx → active/running

**3.** Configuration correct hai

nginx configuration valid

**4.** Files/directories exist karte hain

/etc/nginx/nginx.conf

**5** Port accessible hai

Port 80 → accessible

**6.** Idempotency
Role ko second time run karne par unnecessary changes nahi hone chahiye.

First run  → changed=5
Second run → changed=0
CI mein Molecule kahan aata hai?

Tumhare current workflow mein:

Code Push
   ↓
Jenkins
   ↓
Syntax Check
   ↓
Ansible Lint
   ↓
Check Mode
   ↓
Molecule Tests        ← Step 9
   ↓
CI Result
   ↓
PR Update
Ek line mein interview answer

###### Molecule is a testing framework for Ansible roles that creates a temporary test environment, runs the role, verifies the expected configuration, checks idempotency, and then cleans up the test environment.


----


## Functional test ka matlab hai check karna ki Ansible Role jo kaam karne ke liye banayi gayi hai, woh actual mein sahi kaam kar rahi hai ya nahi.

Simple example

Agar role ka kaam Nginx install aur start karna hai, to functional test check karega:

Nginx installed hai?       → Yes
Nginx running hai?         → Yes
Port 80 accessible hai?    → Yes
Website response de rahi? → Yes

Example command:

systemctl is-active nginx
curl http://localhost

Short definition:

##### Functional testing verifies that the Ansible Role performs its intended functionality correctly.


----

# Ansibkle linting

| Best Practice                       | Simple Meaning                                                               |
| ----------------------------------- | ---------------------------------------------------------------------------- |
| **Use Roles**                       | Related tasks, variables, templates etc. ko proper role structure mein rakho |
| **Use Meaningful Names**            | Har task ko clear `name` do                                                  |
| **Use FQCN**                        | `ansible.builtin.apt` jaise fully qualified module names use karo            |
| **Keep Code Idempotent**            | Role baar-baar run karne par unnecessary changes nahi hone chahiye           |
| **Use Variables**                   | Values ko directly hard-code karne ke bajay variables use karo               |
| **Use Handlers**                    | Service restart jaise actions ko handlers ke through manage karo             |
| **Avoid Unnecessary Shell/Command** | Jahan Ansible module available ho, wahi use karo                             |
| **Use Templates**                   | Dynamic configuration ke liye Jinja2 templates use karo                      |
| **Use Vault for Secrets**           | Passwords, tokens, keys jaise sensitive data ko securely store karo          |
| **Keep Tasks Small**                | Ek task ko ek clear responsibility do                                        |
| **Use Tags Carefully**              | Specific tasks ko selectively run karne ke liye tags use karo                |
| **Test Your Roles**                 | Linting, Molecule aur functional tests se role verify karo                   |

**Interview mein kaise bolna hai?**
Short answer

**Ansible Lint** is a static analysis tool used to check Ansible Playbooks and Roles for syntax-related issues, common mistakes, deprecated practices, and Ansible best practices. It helps improve code quality, consistency, readability, and maintainability.

**Agar interviewer pooche:** "Does Ansible Lint execute the playbook?"

Answer:

No. Ansible Lint primarily performs static analysis of the Ansible code. It does not normally execute the playbook to apply configuration changes.

Agar pooche: "Why do we use Ansible Lint?"

**We use Ansible Lint** to catch coding issues and enforce Ansible best practices before the code is deployed.

Ek line mein yaad rakho:

**Ansible Lint** is like a teacher who checks your Ansible code before you actually run it.

**Haan, Ansible Linting** mein indentation se related YAML formatting/structure issues check ho sakte hain, lekin ek important distinction hai:

**YAML parser/syntax indentation** ko strictly check karta hai, kyunki YAML mein indentation structure define karti hai.
Ansible Lint bhi YAML/Ansible code ko validate karte waqt aise structural issues ko report kar sakta hai.
Lekin Ansible Lint ka purpose sirf indentation check karna nahi hai. Ye best practices, module usage, naming, deprecated patterns, risky code, etc. bhi check karta hai.

Example:

- name: Install nginx
  ansible.builtin.apt:
    name: nginx
    state: present

Yahan indentation structure batati hai:

- name
  └── apt
      ├── name
      └── state

Agar indentation galat ho:

- name: Install nginx
  ansible.builtin.apt:
  name: nginx
  state: present
**ansible-lint** cmd
**ansible-lint playbook.yml**
**ansible-playbook playbook.yml --syntax-check**
**ansible-playbook playbook.yml --check** dry run
