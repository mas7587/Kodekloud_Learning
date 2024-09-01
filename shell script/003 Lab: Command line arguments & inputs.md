## Let's update our script to use a command line variable in the place of lunar-mission


The script is available at the path /home/bob/create-and-launch-rocket. Set the variable mission_name to the command line argument $1


Update the script /home/bob/create-and-launch-rocket as shown below:

    mission_name=$1
    
    mkdir $mission_name
    
    rocket-add $mission_name
    
    rocket-start-power $mission_name
    rocket-internal-power $mission_name
    rocket-start-sequence $mission_name
    rocket-start-engine $mission_name
    rocket-lift-off $mission_name
    
    rocket-status $mission_name

```ruby

launching
bob@caleston-lp10:~$ vi create-and-launch-rocket 
bob@caleston-lp10:~$ bash /home/bob/create-and-launch-rocket 
mkdir: missing operand
Try 'mkdir --help' for more information.
Please input a mission name in the form rocket <mission name>
eg: rocket add lunar-mission
Please input a mission name in the form rocket <mission name>
eg: rocket add lunar-mission
Please input a mission name in the form rocket <mission name>
eg: rocket add lunar-mission
Please input a mission name in the form rocket <mission name>
eg: rocket add lunar-mission
Please input a mission name in the form rocket <mission name>
eg: rocket add lunar-mission
Please input a mission name in the form rocket <mission name>
eg: rocket add lunar-mission
Please input a mission name in the form rocket <mission name>
eg: rocket add lunar-mission
bob@caleston-lp10:~$ cat create-and-launch-rocket 
mission_name=$1

mkdir $mission_name

rocket-add $mission_name

rocket-start-power $mission_name
rocket-internal-power $mission_name
rocket-start-sequence $mission_name
rocket-start-engine $mission_name
rocket-lift-off $mission_name

rocket-status $mission_name
bob@caleston-lp10:~$ bash /home/bob/create-and-launch-rocket 
mkdir: missing operand
Try 'mkdir --help' for more information.
Please input a mission name in the form rocket <mission name>
eg: rocket add lunar-mission
Please input a mission name in the form rocket <mission name>
eg: rocket add lunar-mission
Please input a mission name in the form rocket <mission name>
eg: rocket add lunar-mission
Please input a mission name in the form rocket <mission name>
eg: rocket add lunar-mission
Please input a mission name in the form rocket <mission name>
eg: rocket add lunar-mission
Please input a mission name in the form rocket <mission name>
eg: rocket add lunar-mission
Please input a mission name in the form rocket <mission name>
eg: rocket add lunar-mission
bob@caleston-lp10:~$ 
```

## A shell script /home/bob/print-capital.sh is created for you. When run it currently prints a static message. What's the message you see?

If permissions are not set, please set the correct permissions and execute the script.
```ruby

bob@caleston-lp10:~$ . /home/bob/print-capital.sh 
Capital city of UK is London
bob@caleston-lp10:~$ cat print-capital.sh 
echo "Capital city of UK is London"bob@caleston-lp10:~$ 
```

## Update the script to use 2 command line variables $1 and $2 for country and capital respectively. When the script is executed it should now print the country and its capital using the values passed in as command line arguments.


eg: ./print-capital.sh Nigeria Abuja should print Capital city of Nigeria is Abuja

```ruby
bob@caleston-lp10:~$ . /home/bob/print-capital.sh 
Capital city of UK is London
bob@caleston-lp10:~$ cat print-capital.sh 
echo "Capital city of UK is London"bob@caleston-lp10:~$ vi print-capital.sh 
bob@caleston-lp10:~$ . /home/bob/print-capital.sh india delhi
Capital city of india is delhi
bob@caleston-lp10:~$ cat print-capital.sh 
country=$1
capital=$2

echo "Capital city of $1 is $2"
bob@caleston-lp10:~$ 
```

Update the script to make use of command line arguments:

    echo "Capital city of $1 is $2"

## We have created a new script named /home/bob/backup-file.sh to create a backup of any given file. Update the script to use command line argument $1 for the filename to be backed up instead of the hard-coded filename.


eg: ./backup-file.sh create-and-launch-rocket should backup create-and-launch-rocket to create-and-launch-rocket_bkp

Update the script as shown below:

    # This script creates a backup of a given file by creating a copy as bkp
    # For example some-file is backed up as some-file_bkp
    set -e
    
    file_name=$1
    
    cp $file_name ${file_name}_bkp
    
    echo "Done"

# How many command-line arguments does the script /home/bob/update_shell.sh use?

```ruby
bob@caleston-lp10:~$ cat update_shell.sh 
new_shell="$2"
user_name="$1"
usermod -s  $user_name $new_shellbob@caleston-lp10:~$ 


```

## There is something wrong with the script /home/bob/update_shell.sh. Find out the issue and fix it.


This script should update the bob's home directory and default shell if valid command-line arguments are provided but it is not working correctly.

```ruby
bob@caleston-lp10:~$ cat update_shell.sh 
new_shell="$2"
user_name="$1"
usermod -s  $user_name $new_shellbob@caleston-lp10:~$ 
bob@caleston-lp10:~$ 
bob@caleston-lp10:~$ . /home/bob/update_shell.sh 
usermod: option requires an argument -- 's'
Usage: usermod [options] LOGIN

Options:
  -c, --comment COMMENT         new value of the GECOS field
  -d, --home HOME_DIR           new home directory for the user account
  -e, --expiredate EXPIRE_DATE  set account expiration date to EXPIRE_DATE
  -f, --inactive INACTIVE       set password inactive after expiration
                                to INACTIVE
  -g, --gid GROUP               force use GROUP as new primary group
  -G, --groups GROUPS           new list of supplementary GROUPS
  -a, --append                  append the user to the supplemental GROUPS
                                mentioned by the -G option without removing
                                him/her from other groups
  -h, --help                    display this help message and exit
  -l, --login NEW_LOGIN         new value of the login name
  -L, --lock                    lock the user account
  -m, --move-home               move contents of the home directory to the
                                new location (use only with -d)
  -o, --non-unique              allow using duplicate (non-unique) UID
  -p, --password PASSWORD       use encrypted password for the new password
  -R, --root CHROOT_DIR         directory to chroot into
  -s, --shell SHELL             new login shell for the user account
  -u, --uid UID                 new UID for the user account
  -U, --unlock                  unlock the user account
  -v, --add-subuids FIRST-LAST  add range of subordinate uids
  -V, --del-subuids FIRST-LAST  remove range of subordinate uids
  -w, --add-subgids FIRST-LAST  add range of subordinate gids
  -W, --del-subgids FIRST-LAST  remove range of subordinate gids
  -Z, --selinux-user SEUSER     new SELinux user mapping for the user account

bob@caleston-lp10:~$ vi update_shell.sh 
bob@caleston-lp10:~$ cat update_shell.sh 
new_shell="$2"
user_name="$1"
usermod -s  $new_shell $user_name
bob@caleston-lp10:~$ . /home/bob/update_shell.sh 
usermod: option requires an argument -- 's'
Usage: usermod [options] LOGIN

Options:
  -c, --comment COMMENT         new value of the GECOS field
  -d, --home HOME_DIR           new home directory for the user account
  -e, --expiredate EXPIRE_DATE  set account expiration date to EXPIRE_DATE
  -f, --inactive INACTIVE       set password inactive after expiration
                                to INACTIVE
  -g, --gid GROUP               force use GROUP as new primary group
  -G, --groups GROUPS           new list of supplementary GROUPS
  -a, --append                  append the user to the supplemental GROUPS
                                mentioned by the -G option without removing
                                him/her from other groups
  -h, --help                    display this help message and exit
  -l, --login NEW_LOGIN         new value of the login name
  -L, --lock                    lock the user account
  -m, --move-home               move contents of the home directory to the
                                new location (use only with -d)
  -o, --non-unique              allow using duplicate (non-unique) UID
  -p, --password PASSWORD       use encrypted password for the new password
  -R, --root CHROOT_DIR         directory to chroot into
  -s, --shell SHELL             new login shell for the user account
  -u, --uid UID                 new UID for the user account
  -U, --unlock                  unlock the user account
  -v, --add-subuids FIRST-LAST  add range of subordinate uids
  -V, --del-subuids FIRST-LAST  remove range of subordinate uids
  -w, --add-subgids FIRST-LAST  add range of subordinate gids
  -W, --del-subgids FIRST-LAST  remove range of subordinate gids
  -Z, --selinux-user SEUSER     new SELinux user mapping for the user account

bob@caleston-lp10:~$ cat > update_shell.sh 
new_shell="$2"
user_name="$1"
usermod -s  $new_shell $user_namebob@caleston-lp10:~$ 
bob@caleston-lp10:~$ . /home/bob/update_shell.sh 
usermod: option requires an argument -- 's'
Usage: usermod [options] LOGIN
```
