## Let's update our script to use a variable by the name mission_name.
The script is available at the path /home/bob/create-and-launch-rocket. Update it to define a variable mission_name and set it's value to lunar-mission. Replace all instances of lunar-mission with the new variable $mission_name

Update the file /home/bob/create-and-launch-rocket as shown below:

    mission_name=lunar-mission
    
    mkdir $mission_name
    
    rocket-add $mission_name
    rocket-start-power $mission_name
    rocket-internal-power $mission_name
    rocket-start-sequence $mission_name
    rocket-start-engine $mission_name
    rocket-lift-off $mission_name
    
    rocket-status $mission_name


```ruby
bob@caleston-lp10:~$ cat /home/bob/create-and-launch-rocket 

mkdir lunar-mission

rocket-add lunar-mission

rocket-start-power lunar-mission
rocket-internal-power lunar-mission
rocket-start-sequence lunar-mission
rocket-start-engine lunar-mission
rocket-lift-off lunar-mission

rocket-status lunar-mission
bob@caleston-lp10:~$ vi /home/bob/create-and-launch-rocket 
bob@caleston-lp10:~$ cat > /home/bob/create-and-launch-rocket 
mission_name=lunar-mission

mkdir $mission_name

rocket-add $mission_name
rocket-start-power $mission_name
rocket-internal-power $mission_name
rocket-start-sequence $mission_name
rocket-start-engine $mission_name
rocket-lift-off $mission_name

rocket-status $mission_namebob@caleston-lp10:~$ 
bob@caleston-lp10:~$ 
bob@caleston-lp10:~$ cat /home/bob/create-and-launch-rocket 
mission_name=lunar-mission

mkdir $mission_name

rocket-add $mission_name
rocket-start-power $mission_name
rocket-internal-power $mission_name
rocket-start-sequence $mission_name
rocket-start-engine $mission_name
rocket-lift-off $mission_name

rocket-status $mission_namebob@caleston-lp10:~$ 
```
# A script by the name print-welcome-message.sh is placed in your home directory at /home/bob.


It must print the welcome message with the user name as expected. However it doesn't. Please inspect and fix the problem with the script.



```ruby
rocket-status $mission_namebob@caleston-lp10:~$ cat /home/bob/print-welcome-message.sh 
user_name=Michael

echo "Hi $USER_NAME, Welcome to xFusionCorp Industries. We and the rest of the management are glad to have you on board"
bob@caleston-lp10:~$ vi /home/bob/print-welcome-message.sh 
bob@caleston-lp10:~$ cat /home/bob/print-welcome-message.sh 
USER_NAME=Michael

echo "Hi $USER_NAME, Welcome to xFusionCorp Industries. We and the rest of the management are glad to have you on board"
bob@caleston-lp10:~$ . /home/bob/print-welcome-message.sh 
Hi Michael, Welcome to xFusionCorp Industries. We and the rest of the management are glad to have you on board
bob@caleston-lp10:~$ 
```

Variable name must be in the same case as defined $user_name
## Another script by the name print-uptime.sh is placed in your home directory at /home/bob.


It must print the uptime of the system as expected. However it's not working'. Please inspect and fix the problem with the script.

    Variable name should not have a dashes. Correct it.
```ruby
bob@caleston-lp10:~$ . /home/bob/print-uptime.sh 
-su: up-time=: command not found
The uptime of the system is -time
bob@caleston-lp10:~$ cat /home/bob/print-uptime.sh 
up-time=$(uptime)

echo "The uptime of the system is $up-time"bob@caleston-lp10:~$ cat /home/bob/print-uptimbob@caleston-lp10:~$                                        
bob@caleston-lp10:~$ 
bob@caleston-lp10:~$ vi /home/bob/print-uptime.sh 
bob@caleston-lp10:~$ . /home/bob/print-uptime.sh 
The uptime of the system is  07:25:20 up 17 min,  0 users,  load average: 5.93, 7.54, 12.40
bob@caleston-lp10:~$ cat /home/bob/print-uptime.sh 
uptime=$(uptime)

### Another script by the name backup-file.sh is placed in your home directory at /home/bob.


This script creates a backup of a file by creating a copy of the same file and apending _bkp to it's name. However it's not working'. Please inspect and fix the problem with the script.

    Variable should be encapsulated in { }. Change variable to ${file_name}_bkp

```ruby
bob@caleston-lp10:~$ cat backup-file.sh 
# This script creates a backup of a given file by creating a copy as bkp
# For example some-file is backed up as some-file_bkp

file_name="create-and-launch-rocket"

cp $file_name $file_name_bkp
bob@caleston-lp10:~$ . /home/bob/backup-file.sh 
cp: missing destination file operand after 'create-and-launch-rocket'
Try 'cp --help' for more information.
bob@caleston-lp10:~$ vi backup-file.sh 
bob@caleston-lp10:~$ . /home/bob/backup-file.sh 
bob@caleston-lp10:~$ cat backup-file.sh 
# This script creates a backup of a given file by creating a copy as bkp
# For example some-file is backed up as some-file_bkp

file_name="create-and-launch-rocket"

cp $file_name ${file_name}_bkp
bob@caleston-lp10:~$ ls
backup-file.sh            create-and-launch-rocket_bkp  print-uptime.sh
create-and-launch-rocket  media                         print-welcome-message.sh
bob@caleston-lp10:~$ cat create-and-launch-rocket_bkp 
mission_name=lunar-mission

mkdir $mission_name

rocket-add $mission_name
rocket-start-power $mission_name
rocket-internal-power $mission_name
rocket-start-sequence $mission_name
rocket-start-engine $mission_name
rocket-lift-off $mission_name

rocket-status $mission_namebob@caleston-lp10:~$ 
```

echo "The uptime of the system is $uptime"
bob@caleston-lp10:~$ 
```

### Another script by the name create_files.sh is placed in your home directory at /home/bob.


This script creates files by making use of variables. However, there is something wrong with the script, and not all the files have been created. Please fix it.

```ruby
bob@caleston-lp10:~$ cat create_files.sh 
FILE01="Japan"
FILE02="Egypt"
FILEO3="Canada"

cd /home/bob

echo "Creating file called $FILE01"
touch $FILE01

echo "Creating file called $FILE02"
touch $FILE02

echo "Creating file called $FILE03"
touch $FILE02bob@caleston-lp10:~$ . /home/bob/create_files.sh 
Creating file called Japan
Creating file called Egypt
Creating file called 
bob@caleston-lp10:~$ vi create_files.sh 
bob@caleston-lp10:~$ . /home/bob/create_files.sh 
Creating file called Japan
Creating file called Egypt
Creating file called 
touch: missing file operand
Try 'touch --help' for more information.
bob@caleston-lp10:~$ cat create_files.sh 
FILE01="Japan"
FILE02="Egypt"
FILEO3="Canada"

cd /home/bob

echo "Creating file called $FILE01"
touch $FILE01

echo "Creating file called $FILE02"
touch $FILE02

echo "Creating file called $FILE03"
touch $FILE03
bob@caleston-lp10:~$ vi create_files.sh 
bob@caleston-lp10:~$ . /home/bob/create_files.sh 
Creating file called Japan
Creating file called Egypt
Creating file called Canada
bob@caleston-lp10:~$ 
```

    Make sure that the variable names that are defined are actually the ones being used in the script.


## Inspect and run the script called logged_username.sh. It uses a variable called USER. What is the value assigned to this variable in this script?

```ruby
bob@caleston-lp10:~$ cat logged_username.sh 
echo "My name is $USER"bob@caleston-lp10:~$ . /home/bob/logged_username.sh 
My name is bob
bob@caleston-lp10:~$ 
```
    This script uses an environment variable. If unclear, check out the Bash section in the Linux Basics Course


