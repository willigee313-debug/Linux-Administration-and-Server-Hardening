# Flashcards — Users, Permissions & PAM

Spaced-repetition deck covering account databases, the permission model, special bits, ACLs, sudo/sudoers, `su`/`sg`, and PAM — drawn from the Users, Groups & Permissions module for RHCSA/LFCS/Linux+/LPIC-1 prep.

## Account Databases (/etc/passwd, /etc/shadow)

How many colon-separated fields does /etc/passwd have, and what does field 2 (`x`) indicate?::Seven fields; `x` is a placeholder meaning the password hash lives in /etc/shadow

Which file stores password hashes and password-aging fields, readable only by root?::/etc/shadow

What does `!!` in the /etc/shadow password field mean?::No password has ever been set for the account

What command safely edits /etc/passwd, locking the file and validating on save?::vipw

What command checks /etc/passwd and /etc/shadow for consistency errors?::pwck

## Permissions Model

What octal values correspond to read, write, and execute permissions?::r=4, w=2, x=1

On a directory, what does the execute (x) bit control?::The ability to traverse/enter the directory and access the entries inside it

## Special Permission Bits (SUID/SGID/Sticky)

Which chmod octal digit sets SUID, and what effect does it have on an executable?::4 — the process runs with the file owner's UID (often root) instead of the caller's

What happens when SGID is set on a directory (not a file)?::New files/subdirectories created inside inherit the directory's group instead of the creator's primary group

Which chmod octal digit sets the sticky bit, and what is its canonical real-world use?::1 — protecting a world-writable directory like /tmp so only a file's owner (or root) can delete/rename it

In an `ls -l` listing, what does an uppercase S or T (instead of lowercase s/t) in a special-bit position mean?::The special bit is set but the underlying execute bit is missing

What find command locates every SUID-set file on the system?::find / -perm -4000

Which standard Linux command is the classic example of a SUID-root binary?::/usr/bin/passwd — lets ordinary users change their own password by writing to /etc/shadow

## Access Control Lists (ACL)

What command displays a file's ACL entries?::getfacl

What is the general setfacl syntax to grant a specific user rwx access to a file?::setfacl -m u:username:permissions filename

What character appears at the end of an `ls -l` permission string when a file has ACL entries beyond the base mode bits?::A trailing `+`

## sudo & sudoers

What is the only command that should be used to edit /etc/sudoers?::visudo — it validates syntax before saving

In a sudoers privilege spec, what does a leading `%` before a name denote?::A group rather than an individual user (e.g., %wheel)

Which sudoers tag skips the password prompt for the commands listed after it?::NOPASSWD:

What command lists the sudo commands the current user is authorized to run?::sudo -l

Which sudoers Defaults directive restricts PATH to trusted directories to block PATH-hijack attacks?::secure_path

## su, sg & Real vs Effective UID

What is the key difference between `su user` and `su - user`?::`su - user` starts a full login shell, resetting environment/HOME/PATH to the target user's; bare `su user` keeps the caller's environment

Whose password does `su` prompt for when switching to another account (unless the caller is already root)?::The target account's password

Which command runs a command or shell under one of the invoking user's group memberships?::sg

Which UID — real or effective — governs permission checks for a running process?::Effective UID (EUID)

## PAM (Pluggable Authentication Modules)

What is the per-service PAM configuration directory?::/etc/pam.d/

What are the four PAM management/module types?::auth, account, password, session

In PAM control flags, what does `requisite` do when its module fails?::Aborts the stack immediately and returns failure — no further modules run

Which PAM module enforces password composition rules (length, character classes) at password-change time?::pam_pwquality.so

Which PAM module locks an account after a threshold of consecutive failed logins, and what command resets a locked user?::pam_faillock.so; reset with `faillock --user <name> --reset`

## Related

- [Users, Groups & Permissions](../Users-Groups-and-Permissions/Readme.md) — source module
- [Exam Preparation](Readme.md) — exam hub
- [Linux Administration & Server Hardening](../Readme.md) — course hub
