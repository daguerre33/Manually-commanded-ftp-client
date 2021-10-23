# Manually-commanded-ftp-client
Manually commanded ftp client (Python interactive prompt)

Making this way of connecting to ftp server my aim was to understand how an ftp client works in a low level. You can see the process as you do it step by step.  

There are two channels in the ftp connection: first, the command channel (default port: 21), secondly, the data channel (no default port, the port number is generated by the server again and again), where the server send the data. Therefore you need to create and connect more sockets in order to grab or send a file. 

The commands are binaries (b'...\r\n'), following the standards. The receiving puffer size can be other.

Testing with vsftpd ftp server daemon, you need to give permission for anonymous login.
  $ sudo nano /etc/vsftpd.conf
Change NO to YES on the proper line: 
  Allow anonymous FTP? (Disabled by default).
  anonymous_enable=YES

The test file ('test.txt') is located in /srv/ftp directory, the default directory of vsftpd, with the text: 'this is the content of the test file for testing manually commanded python ftp client\n' (as you will see at the end of the process, below).

Enjoy the process!



#Start python interctive prompt

...@linux:~$ python3
Python 3.6.9 (default, Jan 26 2021, 15:33:00) 
[GCC 8.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.

Import socket modul:
>>> import socket

Make the first socket for commands:
>>> s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

Connect it to the standard ftp command port:
>>> s.connect(('127.0.0.1', 21))

Send username (anonym login user 'ftp'), the output is the number of bytes you sended:
>>> s.send(b'USER ftp\r\n')
10

Send password (anonym login password 'ftp'):
>>> s.send(b'PASS ftp\r\n')
10

Receive feedback:
>>> s.recv(2048)
b'220 (vsFTPd 3.0.3)\r\n331 Please specify the password.\r\n230 Login successful.\r\n'

Make passive mode for getting the data port number generated by the server:
>>> s.send(b'PASV\r\n')
6

Receive feedback with the open port number:
>>> s.recv(2048)
b'227 Entering Passive Mode (127,0,0,1,161,195).\r\n'

Calculate the port number following the received code above:
(127,0,0,1, 161 = x, 195 = y)
Data port number = (x * 256) + y, so:
>>> 161 * 256
41216
>>> 41216 + 195
41411

The open data port's number of the server for sending data is: 41411.
Make the second socket for the data:
>>> q = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

Connect the second socket to the open data port:
>>> q.connect(('127.0.0.1', 41411))

Send command for listing current directory with the first socket (s):
>>> s.send(b'LIST\r\n')
6

Get the data with the second socket(q), a list of the current directory with the test file:
>>> q.recv(4096)
b'
aaa.txt\r\n drwxr-xr-x
...
test.txt\r\n drwxr-xr-x  4 1000 1000 Oct 20 15:01
...
zzz.txt\r\n drwxr-xr-x'

Entering passive mode again for the new data port number:
>>> s.send(b'PASV\r\n')
6
>>> s.recv(1024)
b'227 Entering Passive Mode (127,0,0,1,54,158).\r\n'

Calculate the port number:
>>> 54 * 256
13824
>>> 13824 + 158
13982

Make a new socket, the third (r), for grabbing the content of the file:
>>> r = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

Connect the third socket to the open data port:
>>> r.connect(('127.0.0.1', 13982))

Send command with the first socket (s):
>>> s.send(b'RETR test.txt\r\n')
16

Get data with the third socket (r):
>>> r.recv(4096)
b'this is the content of the test file for testing manually commanded python ftp client\n'

Check whether more data are or not:
>>> r.recv(4096)
b''

If no, close the sockets:
>>> s.close()
>>> q.close()
>>> r.close()

And say hello, vawe good-by:
>>> exit()
