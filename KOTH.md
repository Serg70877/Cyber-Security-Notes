# King Of The Hill

## Owning king.txt file
### chattr
If you give the manual page of chattr binary a read, you'll see that it can set immutable flags on files. That means, even root cannot make mutations in the file without removing that immutable bit.

```bash
chattr +i /root/king.txt
```

This is used by many players to make that king file immutable and hence persisting their name in that file.
To get the upper hand in game, use another bit, 'append-able' on king.txt. This bit makes the file append-able only, and since most of the players 'write' in the file and not append, hence they can't modify the file even though they removed the immutable bit.

### clobber
Now, this is a tricky bit, here, you can set the environment variable setting of root user to prevent overwriting in the files.
Hence, the word clobber, This means that the user cannot add anything to any file using > operator.

Basic command is `set -o noclobber`

But this will only be effective in current shell, so to make it persistent across entire machine, add this to bashrc of root and source that.

## Persistence
### SSH AuthKey
You can always put your ssh keys on to the user/root authorized_keys. So you can always ssh in using them.

> You can use ssh -t to hide your session from tty.

## Defending

### Killing Processes
You can use different commands like `w`, `who`, `ps aux | grep pts` to see who else is on the system so far.

```bash
cat /dev/urandom > /dev/pts/$PTS
```

```bash
kill -9 $PID
```


Taken from https://tryhackme.com/resources/blog/guide-to-king-of-the-hill
