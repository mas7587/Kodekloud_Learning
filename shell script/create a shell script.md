## Let's create our first script. Create a script named create-and-launch-rocket at the path /home/bob and add the below commands to it.


    mkdir lunar-mission
    
    rocket-add lunar-mission
    
    rocket-start-power lunar-mission
    
    rocket-internal-power lunar-mission
    
    rocket-start-sequence lunar-mission
    
    rocket-start-engine lunar-mission
    
    rocket-lift-off lunar-mission
    
    rocket-status lunar-mission


#### Using a text editor such as vi/vim or nano, create a new file called create-and-launch-rocket.

For example, here we are using the VI editor to create the file:

    bob@caleston-lp10:~$ vi /home/bob/create-and-launch-rocket
Next, go to insert mode in VI Editor by pressing the i key and copy paste the each command specified in the question to it:

    mkdir lunar-mission
    
    rocket-add lunar-mission
    
    rocket-start-power lunar-mission
    rocket-internal-power lunar-mission
    rocket-start-sequence lunar-mission
    rocket-start-engine lunar-mission
    rocket-lift-off lunar-mission
    
    rocket-status lunar-mission
Once done, hit the Esc button and type :wq! to save and exit the file.

If you are new to VI Editor, please checkout the Linux Basics Course, which has a detailed introdcution to VI Editor.

## Assign execute permissions to the script

Run the command 

    chmod +x create-and-launch-rocket

# Run your script and make sure lunar-mission is created as expected.


```ruby

bob@caleston-lp10:~$ . /home/bob/create-and-launch-rocket 
mkdir: cannot create directory 'lunar-mission': File exists

--------------------------------------------
          PROJECT lunar-mission           
--------------------------------------------
Creating a new rocket....Done!

Starting power ....Done!

Switching to internal ....Done!

Starting auto sequence ....Done!

Starting engine ....Done!

Initiating lift off ....
        Countdown
          10
          9
          8
          7
          6
          5
          4
          3
          2
          1
          0
   !!!!Lift off!!!
Done!
[53;24HBlast Off!                            ^
                                            / \
                                           /___\
                                          |=   =|
                                          |  K  |
                                          |  O  |
                                          |  D  |
                                          |  E  |
                                          |  K  |
                                          |  L  |
                                          |  O  |
                                          |  U  |
                                          |  D  |
                                         /|##!##|\
                                        / |##!##| \
                                       /  |##!##|  \
                                      |  / ((:)) \  |
                                      | / (((:))) \ |
                                         ((((:)))))
```



Run the command 
    ./create-and-launch-rocket 
    
or 

    bash create-and-launch-rocket

# Let's look at some other general scripts. The script /home/bob/say_hello.sh does not execute. Fix it.


Try running /home/bob/say_hello.sh

Run: 

    chmod +x /home/bob/say_hello.sh
to add execute permissions to the script.

# Create a shell script in the home directory called create-directory-structure.sh. The script should do the following tasks:


    Create the following directories under /home/bob/countries - USA, UK, India
    Create a file under each directory by the name capital.txt
    Add the capital cities name in the file - Washington, D.C, London, New Delhi
    Print uptime of the system
    
Here is a sample solution script:

    mkdir countries
    cd countries
    mkdir USA India UK
    echo "Washington, D.C" > USA/capital.txt
    echo "London" > UK/capital.txt
    echo "New Delhi" > India/capital.txt
    uptime




## We have placed a new version of the same script in your home directory named - create-directory-structure-v2.sh. However there is a problem with it. Inspect the script and identify the cause of the problem and try to fix it.


Tip: This is more of a logical error with the script. While troubleshooting a script, read the script line by line and try to imagine the impact of each command. Imagine the directories and files each command creates/deletes/modifies as well as the present working directory within the script execution.

If you can't figure out the issue easily, another approach is to copy each command within the script line by line and execute it on your shell. This will give you a clear picture of what happens when the script executes.



```ruby

bob@caleston-lp10:~$ cat create-directory-structure-v2.sh 
mkdir countries
cd countries

mkdir USA India UK

echo "Washington, D.C" > countries/USA/capital.txt
echo "London" > countries/UK/capital.txt
echo "New Delhi" > countries/India/capital.txt
```
Check the cd countries line within the script. It changes the PWD to within the countries folder. From there you do not need to specify countries within the path. To solve this, either remove the cd statement, create the countries correct paths and modify the rest of the script to use countries in the path, or remove countries in the path while creating capital.txt files.

Solution 

    mkdir countries
    cd countries
    
    mkdir USA India UK
    
    echo "Washington, D.C" > USA/capital.txt
    echo "London" > UK/capital.txt
    echo "New Delhi" > India/capital.txt
    
    uptime
