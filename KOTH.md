# King Of The Hill

## Owning king.txt file
### chattr
If you give the manual page of chattr binary a read, you'll see that it can set immutable flags on files. That means, even root cannot make mutations in the file without removing that immutable bit.

```bash
chattr +i /root/king.txt
```

This is used by many players to make that king file immutable and hence persisting their name in that file.
To get the upper hand in game, use another bit, 'append-able' on king.txt. This bit makes the file append-able only, and since most of the players 'write' in the file and not append, hence they can't modify the file even though they removed the immutable bit.

Now almost always whoever uses the chattr binary first, either deletes it (foolish move) or hides it somewhere.
Once that's done, you don't have much choice but to either upload your binary or hope that no one deleted busybox from the machine.

```bash
busybox add chattr
```

### clobber
Now, this is a tricky bit, here, you can set the environment variable setting of root user to prevent overwriting in the files.
Hence, the word clobber, This means that the user cannot add anything to any file using > operator.

Basic command is `set -o noclobber`

But this will only be effective in current shell, so to make it persistent across entire machine, add this to bashrc of root and source that.

### cron
You can add your username automatically every minute by putting the following line into `/etc/crontab`
```bash
* * * * * echo "[usernameHere]" > /root/king.txt >/dev/null 2>&1
```

### while loop
```bash
while [ 1 ]; do echo [usernameHere] > /root/king.txt 2>/dev/null; sleep 0.1; done &
```

## Persistence
### SSH AuthKey
You can put your ssh keys in user/root `authorized_keys`, so you can always ssh in using them.

> You can use ssh -t to hide your session from tty.

### adduser
You can use the command `adduser` to add a new user.
Then, you can add the following line into `/etc/sudoers` to give unlimited permissions.
```bash
[usernameHere]  ALL=(ALL:ALL) NOPASSWD:ALL
```

## Defending
You can use different commands like `w`, `who`, `ps aux | grep pts` to see who else is on the system so far.

### Killing Processes
```bash
cat /dev/urandom > /dev/pts/$PTS
```

```bash
kill -9 $PID
```

### Hijacking Terminal
```bash
script /dev/pts/$PTS
```

### Capture PTS Output
Get the process ID of the PTS with
```bash
ps -t pts/$PTS
```

```bash
sudo strace -e write=0,1,2 -e trace=write -s1000 -fp $PID 2>&1 \
| grep --line-buffered -o '".\+[^"]"' \
| grep --line-buffered -o '[^"]\+[^"]' \
| while read -r line; do
    printf "%b" $line;
done
```

### Removing Persistence
- Comment out `/etc/password`
- Remove authorized_keys in `.ssh` folder

Taken from https://tryhackme.com/resources/blog/guide-to-king-of-the-hill
