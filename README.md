# Operating Systems Security — Assignment 3

**Fall 2024 — Team E**
*A hands-on lab covering Linux ACLs and manual binary exploitation (stack overflow, optimization effects, ROP, and mitigations).*

## Project overview

This repository documents our Assignment 3 work for the Operating Systems Security course. The goals:

* **Access Control Lists (ACLs):** create users `test1`..`test5`, directories `dir1` and `dir2`, and files `file1`..`file6`. Apply ACL rules to meet the assignment requirements and document them.
* **Binary exploitation:** analyze, test and (safely, in an isolated VM) exploit a vulnerable SUID binary `auth` to study stack overflows, compiler optimization effects, ROP, and mitigation techniques (stack canaries, ASLR/PIE, NX).

Everything here is educational. Run experiments only in an isolated, revertible VM and with instructor approval.

---

## How to reproduce (HIGH LEVEL — run only in isolated VM)

### Build (examples)

```bash
# O0 (unoptimized, debug)
gcc -g -Wa,-adlhn=auth0.s -O0 -o auth0 auth.c -fverbose-asm -masm=intel -lcrypt
sudo chown root:root auth0
sudo chmod u+s auth0

# O3 (optimized)
gcc -g -Wa,-adlhn=auth3.s -O3 -o auth3 auth.c -fverbose-asm -masm=intel -lcrypt
sudo chown root:root auth3
sudo chmod u+s auth3
```

### Debugging example

```bash
# Start gdb and break at main
gdb -q auth0 -ex "b main" -ex "r"

# Test large input (ONLY inside VM)
gdb -q auth -ex "b main" -ex "r $(python3 -c 'print(\"A\"*512)')"
```

### ACL examples (run as root in the lab VM)

```bash
# create test users
for i in 1 2 3 4 5; do sudo useradd -m test$i; done

# make directories and files
sudo mkdir -p /home/lab/dir1 /home/lab/dir2
sudo touch /home/lab/dir1/file1 /home/lab/dir1/file2 /home/lab/dir1/file3
sudo touch /home/lab/dir2/file4 /home/lab/dir2/file5 /home/lab/dir2/file6

# example ACL (give test2 read-only on dir1 and its files)
sudo setfacl -m u:test2:rx /home/lab/dir1
sudo setfacl -m u:test2:r /home/lab/dir1/file1
# Full set of ACL commands is in acl/rules.acl
```

---

## Deliverables (what to include when handing in)

1. `acl/rules.acl` or `acl/commands.txt` — exact `setfacl/getfacl` commands used, with brief explanations for each rule (how they meet the objectives for test1..test5).
2. `gdb-logs/` — annotated GDB transcripts showing the commands you ran and your observations.
3. `asm/` — `auth0.s` and `auth3.s` with quoted assembly lines that changed behaviour and a short explanation.
4. `exploits/` — ROP strategy and payload structure (or pseudocode if actual exploit code is disallowed).
5. `reports/final_report.pdf` — combined write-up with:

   * Explanation of `auth.c` internals, libraries used, and external calls.
   * Full list of GDB commands used during the overflow analysis and explanation of observations.
   * ROP attempt details and gdb output demonstrating results (or justification why not included).
   * Mitigation experiments and conclusions (`-fstack-protector-all`, ASLR/PIE, NX).
6. `slides/` — short presentation (optional).

---

## Safety & best practices (read before running)

* **Always** run vulnerable binaries in an isolated VM with snapshots you can revert.
* Do **not** expose lab VMs to untrusted networks while testing SUID binaries or exploits.
* Get instructor approval before running or sharing any exploit code.
* Document every destructive action (e.g., changing file owners or permissions) and revert changes when done.

---

## Quick checklist before submission

ACL rules implement objectives (test1..test5).
GDB logs show stack memory, argv, and crash/overflow behavior.
Assembly comparison highlights changed control flow between `-O0` and `-O3`.
ROP strategy and offsets are documented and validated in gdb (or pseudocode provided).
Tests for `-fstack-protector-all`, ASLR, PIE, and stack executability performed & explained.

---

## Team

Team E — contributors:

* Hadeer Amr
* Farida Amed
* Yossef Ahmed
* Ahmed Yasser
* Yossef Tamer
* Hana Emad
* Martin Maher
* Malak Wissam

---

## License & ethics

This work is educational. Suggested license: **MIT**. Use it ethically — experiments should only be performed in controlled lab environments and with permission.

---
