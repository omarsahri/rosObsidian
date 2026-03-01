# Why `sudo` did not work immediately after adding the user

## What happened
You added `user` to the `sudo` group while you were in a `root` shell, and the command confirmed that the account was a member of `sudo`.

However, when you went back to your existing `user` shell and ran:

```bash
sudo apt update
```

you still got:

```text
user is not in the sudoers file
```

Later, after running:

```bash
su - user
```

and testing again, `sudo` started working.

## Why this happened
Adding a user to the `sudo` group updates the system's group database, but an already-running shell session may still be using the old group membership information.

So even though the account had already been added correctly, the shell you returned to was still acting like `user` was **not** in the `sudo` group yet.

In other words:

- the **system** knew `user` was in `sudo`
- the **current shell session** had stale group information

## Important distinction
In `/etc/sudoers`, you usually do **not** see the username directly.
You see a rule like this:

```text
%sudo ALL=(ALL:ALL) ALL
```

That means:

- any user in the `sudo` group can use `sudo`
- the username does not need its own separate line in `/etc/sudoers`

So the real question is not:

> "Do I see `user` inside `/etc/sudoers`?"

It is:

> "Is `user` currently in the `sudo` group, and has the current shell reloaded that membership?"

## Why `su - user` fixed it
This command starts a **fresh login shell** for `user`:

```bash
su - user
```

That new login shell reloads group membership from the system.
Once that happened, the shell recognized that `user` belongs to `sudo`, so commands like:

```bash
sudo whoami
```

started working.

## How to verify this next time
### Check the current shell's groups
Run:

```bash
id
groups
```

If `sudo` is missing there, the current shell has not picked up the new group membership.

### Check the actual account membership from root
Run:

```bash
id user
getent group sudo
grep '^sudo:' /etc/group
```

If `user` appears there, the add worked correctly.

## Clean ways to refresh the session
Any of these can fix it:

```bash
su - user
```

or fully log out and log back in.

Sometimes this also works:

```bash
newgrp sudo
```

## Final takeaway
Nothing was wrong with the `sudoers` configuration.
The issue was that the **old shell session had stale group membership**, and `su - user` created a fresh login session that reloaded the correct groups.
