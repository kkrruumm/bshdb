# bshdb
bshdb - "Basic Shell Database"
This is a simple shell script database library intended for use by other shell scripts.

This is inherently fairly slow, primarily when creating database entries. This is not suitable for large databases or places where performance is the primary concern.

# Q/A


Q: Should I use this?

A: Probably not.

---

Q: Wait, the database files are shell scripts too?

A: Yeah. Kinda cool... right?

---

Q: Why did you write this?

A: For fun. But, also, I wanted a way to write a database to a single file using utilities that already exist on my systems.

---

Q: Is this vulnerable to shell injection?

A: Yes, inherently. I suggest addressing the root cause of this issue, that being malware on your system, or using something more reasonable than this for your database. At some point I may look into a way to verify the database contents to prevent this.

---

Q: Why do you store the entire database file in RAM during manipulation?

A: Due to limitations of CLI text editors that are properly scriptable, I've chosen to use sed (explicitly without -i) to manipulate the database. This will absolutely be awful if you have an unreasonably large database.

A part 2: Since sed generally creates a temporary file on the disk, this is a problem for certain applications that are a bit more data sensitive. If you were to have an encrypted file that you wish to use for your database, it would be spilling unencrypted contents onto the disk. To avoid this, this runs sed without `-i`, and contains its outputs inside of a variable to keep it in RAM.

# Dependencies

```
gnu sed
grep
head
cut
cat

Coreutils in general, whether that's gnu or something like busybox.
```

# Usage

This database library follows the structure of

database file -> entries -> fields

The database file will contain entries, which contain fields that hold the actual values.

This database file needs to be sourced in the shell script that is using it, this can be done by adding the following to your script:

```
. /path/to/bshdb
```

Once sourced, you have access to quite a few commands:

Creating a database file:
```
create_database /path/to/database/file
```
Deleting a database file:
```
delete_database /path/to/database/file
```
Creating a database entry:
```
create_database_entry /path/to/database/file nameofentry
```
Creating a database entry field (The variables that actually hold information in entries):
```
create_database_entry_field /path/to/database/file nameofentry nameoffield
```
Deleting a database entry:
```
delete_database_entry /path/to/database/file nameofentry
```
Deleting a database entry field:
```
delete_database_entry_field /path/to/database/file nameofentry nameoffield
```
Listing database entries (echoes):
```
get_database_entries /path/to/database/file
```
Listing database entry fields (echoes):
```
get_database_entry_fields /path/to/database/file nameofentry
```
Setting a database entry field value:
```
set_database_field_value /path/to/database/file nameofentry nameoffield fieldvalue
```
Outputting a database entry field value (echoes):
```
get_database_field_value /path/to/database/file nameofentry nameoffield
```

A successful bshdb command should always return 0, any failure whatsoever should return 1.

This means that you can execute your commands and verify their functionality at the same time via something like:

```
if create_database /path/to/database/file ; then
    # Do stuff here
fi
```

# Notes

This database library *is* pretty slow when creating entries.

I've benchmarked this with the following script to create 1000 database entries:

```
. /path/to/bshdb

i=0

while [ "$i" -ne 1000 ];
do
    create_database_entry /path/to/database/file a"$i"
    i=$(($i+1))
done
```

On my system (i7 10700k, 32gb 3200mhz cl16 RAM, Samsung 970 evo plus NVME + XFS), this benchmarked as follows with gnu coreutils:
```
$ time ./controlscript 

real	0m4.054s
user	0m3.079s
sys	0m1.106s
```

When actually manipulating this database aside from entry creation, it is performant enough to not be an issue for anything you would actually want to use this for.
