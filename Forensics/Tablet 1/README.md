# Challenge Description

<img src="images/img1.png" />
<br>

We're given a **Mobile Forensics** challenge which can be identified as an `iPad` Dump.
There are some handy tools in order to analyze iOS Mobile dumps, such as:
* iLEAPP (https://github.com/abrignoni/iLEAPP)
* mac_apt (https://github.com/ydkhatri/mac_apt)

If we look into the challenge's description:
```
Red has been acting very sus lately... so I took a backup of their tablet to see if they are hiding something!
It looks like Red has been exfiltrating sensitive data bound for Mira HQ to their own private server. 
We need to access that server and contain the leak.
```

We can make a hypoteses where there are some backup of an `apps` where we can make further analysis.
There are reference to `private server` and **we need to access that server**. 
Second hypotese -> Remote connections are needed.
My first assumption lists:
* `netcat - related`
* `SSH - related`
* `telnet/ftp`

# Analysis

If we extract the file `tablet.tar.gz`, we can find something "phishy" in the folder:
`private\var\root\.bash_history` where there's a bash history command that was logged:

```bash
ls
exit
tar --version
exit
cd ../mobile/Containers/Data/Application/
find ./ -iname *hammerandchisel* -type d 2>/dev/null
cd 0CE5D539-F72A-4C22-BADF-A02CE5A50D2E/
ls
cd Library/
ls
cd Caches/
ls
cd com.hammerandchisel.discord/
ls
rm -rf *
ls
cd ..
ls
ls
cd com.hammerandchisel.discord/
ls
exit
cd ../mobile/Containers/Data/Application/AA7DB282-D12B-4FB1-8DD2-F5FEF3E3198B/Library/Application\ Support/
rm webssh.db 
exit
```

These commands are going to be crucial for the next challenge as well. We might see something interesting in
`webssh.db` since it was removed, but we can still find the database which located in
`private\var\mobile\Containers\Data\Application\AA7DB282-D12B-4FB1-8DD2-F5FEF3E3198B\Library\Application Support`

We can open the `.db` file in SQLite Browser (https://sqlitebrowser.org/).
The database structure consists of 2 tables, `NSFKey` and `NSFValues`.
<img src="images/img2.png" />

<br>

However, `NSFValues` Table has some interesting columns which contains some SSH Confidential Information.
We can retrieve information from there:

<img src="images/img3.png" />
<br>

`privatePart` which contains SSH Private Key.
```
-----BEGIN OPENSSH PRIVATE KEY-----
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
9HmS+uFZxtacdqj4KdrHPcagm3Q=
-----END OPENSSH PRIVATE KEY-----
```

`host` contains Private Domains/Hosts -> `red.chal.uiuc.tf`

We also get the SSH Username's Login -> `red`
and a custom port in order to connect -> `42069`

Finally, we have the `decryptedPassword` which is `********`

We can proceed to connect via SSH:
`ssh -i private_key red@red.chal.uiuc.tf -p 42069`

Yet:

<img src="images/img4.png" />
<br>

We can't use SSH as SFTP is the only connections that are allowed, so
we have to switch to `SFTP`

<img src="images/img5.png" />

Surfing through the directories and files, leads to an interesting findings
from `/bash_history`:

```bash
sftp> ls -la
dr-xr-xr-x    1 red      red          4096 Jul 29 02:23 .
drwxr-xr-x    1 root     root         4096 Jul 29 02:23 ..
-r-xr-xr-x    1 red      red            31 Jul 25 14:28 .bash_history
-r-xr-xr-x    1 red      red           220 Feb 25  2020 .bash_logout
-r-xr-xr-x    1 red      red          3771 Feb 25  2020 .bashrc
-r-xr-xr-x    1 red      red           807 Feb 25  2020 .profile
dr-xr-xr-x    1 red      red          4096 Jul 29 02:23 .ssh
sftp> pwd
Remote working directory: /home/red
sftp> cd ..
sftp> pwd
Remote working directory: /home
sftp> ls -la
drwxr-xr-x    1 root     root         4096 Jul 29 02:23 .
drwxr-xr-x    1 root     root         4096 Jul 31 15:22 ..
dr-xr-xr-x    1 red      red          4096 Jul 29 02:23 red
sftp> ls -la
drwxr-xr-x    1 root     root         4096 Jul 29 02:23 .
drwxr-xr-x    1 root     root         4096 Jul 31 15:22 ..
dr-xr-xr-x    1 red      red          4096 Jul 29 02:23 red
sftp> pwd
Remote working directory: /home
sftp> cd ..
sftp> grep -r -i "RTL"
Invalid command.
sftp> ls
bin     boot    dev     etc     home    kctf    lib     lib32   lib64   
libx32  media   mnt     opt     proc    root    run     sbin    srv     
sys     tmp     usr     var     
sftp> cd tmp
sftp> dir
sftp> cd var
sftp> ls
backups  cache    lib      local    lock     log      mail     opt      
run      spool    tmp      
sftp> cd 
sftp> pwd
Remote working directory: /home/red

sftp> ls -la
dr-xr-xr-x    1 red      red          4096 Jul 29 02:23 .
drwxr-xr-x    1 root     root         4096 Jul 29 02:23 ..
-r-xr-xr-x    1 red      red            31 Jul 25 14:28 .bash_history
-r-xr-xr-x    1 red      red           220 Feb 25  2020 .bash_logout
-r-xr-xr-x    1 red      red          3771 Feb 25  2020 .bashrc
-r-xr-xr-x    1 red      red           807 Feb 25  2020 .profile
dr-xr-xr-x    1 red      red          4096 Jul 29 02:23 .ssh
sftp> get .bash_history
Fetching /home/red/.bash_history to .bash_history
/home/red/.bash_history                 100%   31     0.1KB/s   00:00    
```

The content:
```bash
cat .bash_history
mv /srv/exfiltrated "/srv/..."
```

There is `exfiltrated` directories which renamed as `...`. 
We proceed to see what's behind the directories and we found `important_data.jpg` which
leads to the flag.

```bash
sftp> cd srv
sftp> ls
sftp> ls -la
drwxr-xr-x    1 root     root         4096 Jul 29 02:23 .
drwxr-xr-x    1 root     root         4096 Jul 31 15:22 ..
dr-xr-xr-x    1 root     root         4096 Jul 29 02:23 ...
sftp> cd ...
sftp> ls -la
dr-xr-xr-x    1 root     root         4096 Jul 29 02:23 .
drwxr-xr-x    1 root     root         4096 Jul 29 02:23 ..
-r-xr-xr-x    1 root     root        43621 Jul 25 14:28 important_data.jpg
sftp> get important_data.jpg 
Fetching /srv/.../important_data.jpg to important_data.jpg
/srv/.../important_data.jpg             100%   43KB  42.6KB/s   00:01    
sftp>
```
<br>

<img src="images/important_data.jpg" />


