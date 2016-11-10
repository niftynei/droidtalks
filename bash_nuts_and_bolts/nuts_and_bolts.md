^ this is a  talk about shells. a basic intro to shells, rather.

# shells shellmoji ??  

# fun things to cover

- where do i find a shell?
  -> open up terminal...
- what is a shell?
  -> open & say
- shell as a programming environment
  -> variables 
  -> method calls w/ parameters
  -> flags
  -> getting help
  -> .bashrc

- What is a PATH?
- What does `export` do?

- windows & tabs ?? (when does .bashrc vs .bash_profile get initialized?)
- some shell-y shortcuts

the file system??

-------

# what is a shell?
Shell is a programming environment that terminal runs. it's a text based GUI for interacting with your system.

# what is shell good for?
shell can give you information about your system, allows you to modify and run programs. let's try something

`say "hello lisa"`
`open http://recurse.com`

#what is a command? 
a command is a bash program, often referred to as a bash script. let's make a bash script really quickly.

    #!/bin/sh

    say "hello lisa"

Move this into your /usr/local/bin
Try to run & fail -- change the permissions to the wrong thing.

   chmod 755 /usr/local/bin/sayhello

accidentally change the permissions on the wrong folder.

// use history to see what i actually typed in

we can observe the permiessions a file currently has by listing all the flles in a directory (ls),

   ls -lah /usr/local/bin

let's look and see what rwx stands for.. (scroll to portion on MoDES)
   
   man chmod

#what are -lah for?
-lah are flags. let's see what happens when we only pass in a few of them...


# what commands are available in shell?
bash looks for executable files in certain directories; executable files in these directories become indexed and available as 'first level' commands. (i just made up that bit about being first level)

bash is a programming environment; it stores the set of directories in a variable called PATH. we can see this PATH by printing it to stdout.

   echo $PATH`

we can see what all of the environment variables are by running `printenv` or `env`

HELLO="this is hello"

echo $HELLO; echo HELLO


# where did all those other things in my path come from?
as an android developer, i wanted to be able to add the set of Android tools to my path.

it's possible to configure these variables, and set new ones for your shell environment in a configuration file.

// show off .bashrc && explain difference between ,bashrc and .bash_profile

// remove android utilities from path to show how that works, open a new terminal & explain why i have to open a new terminal


# talk about pipes and stdout?
'chain of command' 

// show off cat and pipe to wc. use pbcopy and pbpaste for this.
// show off wc using pbcopy on some text from a website
// redirect some text into a file

// pretty good site for redirection http://www.catonmat.net/blog/bash-one-liners-explained-part-three/

# talk a little bit about processes?
sometimes you want to run a long running process or command but still be able to do other things in the shell. 
you can run shells inside shells, or fork off processes from a shell.

// go to static site (~/dev/website). cat preview and then run it.
// run site in background (by adding a & to the end of it)
// show all jobs running with `jobs` terminate job with `kill %ID`
// show off `top` and `ps -ax`

# net cat : networking and redirection
http://osxdaily.com/2014/03/27/send-data-across-network-netcat/

    nc -k -l 2999 

// get your ip address    
ipconfig getifaddr en0

// send data from another machine
cat sendthisdataover.txt | nc $IP_ADDRESS 2999
pbpaste | nc $IP_ADDRESS 2999


# some things we didn't talk about
moving, copying files
monitoring disk space or memory usage
tips and tricks; shortcuts

PLAY BANDITT!
http://overthewire.org/wargames/bandit/

-----

^bash is a programming environment that is directly hardwired to your file system... in my mind liek being wired into the matrix directly. it provides a pre-defined set of commands (think single method programs) that can do a host of computer and file maintenance tasks.  you can interact with programs 


^ When a program is invoked it is given an array of strings called the environment. This is a list of name-value pairs, of the form name=value

^ Exported variables get passed on to child processes, not-exported variables do not.

---- 

PLAY BANDIT! http://overthewire.org/wargames/bandit/
