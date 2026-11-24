# Mock Exam Handoff Template — What to Give the Assistant for Direction

Use this during **mock exams only** (AI chatbots are prohibited in the real OSCP+ exam and reporting phase).
Fill in the sections below as you go, paste them into your Notion page, and ping the assistant at natural
decision points ("scan done, choose my first target", "stuck on privesc for host X", "which AD edge first").
The more of this that is present, the better the direction.

**Golden rule: paste text for anything textual, screenshots only for things that are inherently visual**
(graphs, GUIs). Text can be read exactly; screenshots of terminals lose detail.

---

## 0. The one-line ask format

Every check-in should end with a short block like this:

```
STATE: <time elapsed>/<23h45 mock window>
TARGET: <host name or IP>
WHAT I KNOW: <2–4 bullets of confirmed facts>
WHAT I TRIED: <commands/approaches + why they failed>
MY HYPOTHESIS: <what you think the path is, if any>
ASK: <specific> e.g. "which of these 3 leads first?" / "what enumeration gap?" / "hint only, not the answer"
```

Telling the assistant your constraint ("hint only", "no full solution") keeps the mock realistic.

---

## 1. Global session info (once, at the start)

| Item | Value |
|---|---|
| Mock platform (PG Practice / HTB / own lab / Cauldron / etc.) | |
| Exam-like structure? (3 standalones + AD set yes/no) | |
| Mock window length (23h45 or shorter practice) | |
| Your Kali IP (tun0/VPN IP) | |
| Your experience level honestly stated (helps calibrate hints) | |
| What you want from the session (direction vs. teaching vs. full steps when truly stuck) | |

---

## 2. Per standalone box — what to provide

### 2.1 Before you have a shell (enumeration phase)

Text to paste (not screenshot):

1. **Open ports list** from the full TCP scan:
   `grep -oE '[0-9]+/open/tcp' scans/nmap/full_tcp.gnmap | cut -d/ -f1 | sort -n -u | paste -sd, -`
2. **Full `nmap -sCV` output** on those ports, verbatim (`sudo nmap -sCV -Pn -p "$ports" -oA scans/nmap/services "$IP"` then `cat scans/nmap/services.nmap`)
3. **UDP results** if you ran them (top-20 scan output)
4. **whatweb output** for every web port: `whatweb -a 3 http://$IP:$PORT | tee scans/web/whatweb_$PORT.txt`
5. **`curl -i` of the web root** and of robots.txt on every web port
6. **Content discovery results**: the exact feroxbuster/gobuster command + the interesting hits only
   (200/301/403 with sizes — not the full 10,000-line output)
7. **Findings from the app itself**: endpoint list, parameters, forms, login points, HTML comments,
   noted versions/CMS from page footers or /etc/passwd-style leaks
8. **Every credential/user/key found so far** and its source

Screenshots (for inherently visual things):

9. **Browser view of every interesting page** (login, upload, admin panel, unusual error)
10. **Burp Proxy HTTP history** filtered view, showing URLs and sizes
11. **Burp request + response pair** for each input you tested and what happened
    (Repeater "save item" or plain screenshot with both panes visible)

### 2.2 After you get a shell

Text to paste:

1. **Identity + hostname**: `id` / `whoami; whoami /priv` (per OS), `hostname`, `ip addr` / `ipconfig`
2. **On Linux**: `sudo -l`, SUID list (`find / -perm -4000 -type f 2>/dev/null`), cron jobs,
   `/etc/passwd` interesting users, listening ports (`ss -tlnp`), LSE/linpeas output **triaged**
   (paste the red/yellow items, not the full run)
3. **On Windows**: `whoami /all`, `systeminfo`, `net localgroup administrators`, unquoted service paths
   (`sc qc` output), scheduled tasks, PrivescCheck output **triaged** (FINDING lines only)
4. **What the user can access** that others cannot: readable DB configs, service account data,
   interesting environment variables, cached credentials
5. local.txt captured? (its existence, not necessarily its value)
6. **Your privesc hypotheses so far and why they failed**

### 2.3 Where exactly you're stuck

- The last 3–5 commands you ran and their outputs
- The error message you don't understand (paste verbatim)
- What the walkthrough/cheat sheet step said, and where it stopped matching reality

---

## 3. The AD set — what to provide

### 3.1 Credential validation + domain basics (text)

1. **Domain name, DC hostname, DC IP**, all hosts in the set with hostnames/roles
2. **The exact nxc outputs**: credential validation (`nxc smb/ldap/winrm`), and enumeration
   (`--users --groups --pass-pol --shares --sessions`) — paste the outputs verbatim
3. Which hosts accepted the supplied credential, and which showed `(Pwn3d!)`
4. **SYSVOL/GPP check**: did you grep cpassword, and what was readable in shares?
5. Any usernames/descriptions/group memberships that look like leads

### 3.2 BloodHound (this is where screenshots beat text)

Paste as text:

1. **Database Info node counts** (users/computers/groups/sessions) — to confirm collection worked
2. The list of **principals you marked as Owned** and why
3. **Kerberoast/AS-REP results**: which accounts are roastable, whether hashes cracked, the etype

Screenshots:

4. **Shortest Path from your owned user to Domain Admins** — full graph screenshot with the edge
   labels readable (zoom in; the assistant needs the exact edge names)
5. **Node info panel** for the princip*al you're considering attacking — the outbound control
   edges (GenericAll/WriteDacl/etc.) must be legible
6. **Any unusual graph pattern** you want explained (screenshot + "what does this arrow mean?")

### 3.3 After each hop

- New user's group memberships (`whoami /groups`), what's now accessible that wasn't
- Re-run SharpHound as the new principal — and tell the assistant you re-collected

---

## 4. How to capture all this efficiently (so it costs you ~5 minutes, not 50)

**Text capture — do this from the start, it auto-builds your handoff:**

```bash
# Every command, teed into a per-target log (do this reflexively):
nxc smb $IP -u u -p p --shares 2>&1 | tee -a OSCP/$IP/notes_raw.log

# Or capture a whole session including prompts:
script -a OSCP/$IP/session.log       # start once; Ctrl-D to stop
# tmux alternative for one pane's history:
tmux capture-pane -p -S -1000 > OSCP/$IP/pane_dump.txt
```

**Screenshot capture:**

```bash
# Kali: use the Print key, or:
flameshot gui        # select region, annotate, copy
scrot -u '%H_%M_window.png'    # quick window grab
```

**Burp**: right-click any request → "Save item" (text file) beats a screenshot; screenshot only for
the full Repeater view when you want to discuss both panes at once.

**Naming discipline** (this is what makes the Notion page usable by both of you):

```
<target>/<phase>_<tool>_<n>.<ext>
app01/enum_feroxbuster_1.png
app01/access_burp_login_2.png
dc01/bh_pathtoda_d.png
```

**The 60-second handoff**: before you ping, copy (a) the relevant section of your notes.md state
line, (b) the last ~20 lines of the relevant log, (c) 1–2 screenshots max. Do not spend exam
minutes producing anything prettier than that.

---

## 5. What you'll get back (so you can calibrate the ask)

- **Direction when you ask for direction**: which lead to work first, what enumeration gap exists,
  and the reasoning — the two commands to run next, not the flag.
- **Verification when you paste findings**: confirmation of your read of the output, corrections
  where you misread (very common with nmap/kerberos/hash formats), and what the finding implies.
- **Teaching when you ask**: explanations of any unfamiliar concept the output mentions.
- **Stuck-rescue when you ask last-resort**: a structured nudge that still leaves the exploit to you.
- Ask to keep the mock realistic ("hint ladder"): first response = nudge; escalate only if you ask.

---

## 6. Health warnings (things that make handoffs fail)

1. Terminal screenshots instead of pasted text = lost detail and slow rounds. Paste text.
2. "It didn't work" with nothing attached = unactionable. Always: command + verbatim output.
3. Forgetting to say what you already tried = you get told to do it again.
4. Sending 30 screenshots = nobody finds the signal; send the 2 that matter.
5. Not re-collecting BloodHound after a new credential = the assistant reasons about a stale graph.
6. Treat the assistant's direction as a hypothesis to verify, not a fact — you are the one building
   the skill; on the real exam day this document does not exist.
