#!/usr/bin/env python3
import os
import socket
import sys

help_text = """usage: 3700ftp.py [-h] [--verbose] operation params [params ...]

FTP client for listing, copying, moving, and deleting files and directories on remote FTP servers.

positional arguments:
operation      The operation to execute. Valid operations are 'ls', 'rm', 'rmdir',
              'mkdir', 'cp', and 'mv'
params         Parameters for the given operation. Will be one or two paths and/or URLs.

optional arguments:
-h, --help     show this help message and exit
--verbose, -v  Print all messages to and from the FTP server

# Available Operations

This FTP client supports the following operations:

ls <URL>                 Print out the directory listing from the FTP server at the given URL
mkdir <URL>              Create a new directory on the FTP server at the given URL
rm <URL>                 Delete the file on the FTP server at the given URL
rmdir <URL>              Delete the directory on the FTP server at the given URL
cp <ARG1> <ARG2>         Copy the file given by ARG1 to the file given by
                          ARG2. If ARG1 is a local file, then ARG2 must be a URL, and vice-versa.
mv <ARG1> <ARG2>         Move the file given by ARG1 to the file given by
                          ARG2. If ARG1 is a local file, then ARG2 must be a URL, and vice-versa.

# URL Format and Defaults

Remote files and directories should be specified in the following URL format:

ftp://[USER[:PASSWORD]@]HOST[:PORT]/PATH

Where USER and PASSWORD are the username and password to access the FTP server,
HOST is the domain name or IP address of the FTP server, PORT is the remote port
for the FTP server, and PATH is the path to a file or directory.

HOST and PATH are the minimum required fields in the URL, all other fields are optional.
The default USER is 'anonymous' with no PASSWORD. The default PORT is 21.

# Example Usage

List the files in the FTP server's root directory:

$ ./3700ftp ls ftp://bob:s3cr3t@ftp.example.com/

List the files in a specific directory on the FTP server:

$ ./3700ftp ls ftp://bob:s3cr3t@ftp.example.com/documents/homeworks

Delete a file on the FTP server:

$ ./3700ftp rm ftp://bob:s3cr3t@ftp.example.com/documents/homeworks/homework1.docx

Delete a directory on the FTP server:

$ ./3700ftp rmdir ftp://bob:s3cr3t@ftp.example.com/documents/homeworks

Make a remote directory:

$ ./3700ftp mkdir ftp://bob:s3cr3t@ftp.example.com/documents/homeworks-v2

Copy a file from the local machine to the FTP server:

$ ./3700ftp cp other-hw/essay.pdf ftp://bob:s3cr3t@ftp.example.com/documents/homeworks-v2/essay.pdf

Copy a file from the FTP server to the local machine:

$ ./3700ftp cp ftp://bob:s3cr3t@ftp.example.com/documents/todo-list.txt other-hw/todo-list.txt"""

# default port and hostname
port = 21
hostname = "ftp.3700.network"

# username musserca
# password "54eQhaVBNnlSmvjyJA1O"

# command line arguments
argument_list = sys.argv[1:]

# misc global variables
verbose = False
BUFFER_SIZE = 4096

# command parse initial
if len(argument_list) < 2:
    print("Not enough arguments :(\n")
if argument_list[0] == "--help" or argument_list[0] == "-h":
    print(help_text)
elif argument_list[0] == "-v" or argument_list[0] == "--verbose":
    verbose = True
    argument_list = argument_list[1:]

# initialize socket control
sock_control = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_address = (hostname, port)
sock_control.connect(server_address)

# read hello message

print(sock_control.recv(1024).decode("utf-8"))


# methods

# send data through control and return result
def send_socket_control(data):
    sock_control.sendall(bytes(data, encoding="utf-8"))
    received_data = sock_control.recv(1024)
    received_data = received_data.decode("utf-8")
    return received_data


# process the ls command by parsing path and sending via control
def lsCommand(url):
    # path parse
    ar = url.split("/")
    data = "LIST /" + ar[3] + "\r\n"
    print(send_socket_control(data))


# process the retr command by parsing path and sending via control
def retrCommand(url):
    # path parse
    ar = url.split("/")
    data = "RETR /" + ar[3] + "\r\n"
    print(send_socket_control(data))


# process the store command by parsing path and sending via control
def storCommand(url):
    # path parse
    ar = url.split("/")
    data = "STOR /" + ar[3] + "\r\n"
    print(send_socket_control(data))


# login through control socket
def login(u, p):
    print(send_socket_control("USER " + u + "\r\n"))
    if p != "":
        print(send_socket_control("PASS " + p + "\r\n"))


# ask the server to set up a data channel and return the ip and port
def pasv():
    res = send_socket_control("PASV\r\n")
    print(res)
    arr = res[27:len(res) - 4].split(",")
    ip = arr[0] + "." + arr[1] + "." + arr[2] + "." + arr[3]
    port1 = str((int(arr[4]) << 8) + int(arr[5]))
    return ip, port1


# create the data socket based off the ip and port given
def create_data_socket(ip_given, port_given):
    data_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_address_data = (ip_given, int(port_given))
    data_socket.connect(server_address_data)
    return data_socket


# do the required downloading and uploading mode changes
def download_upload():
    print(send_socket_control("TYPE I\r\n"))
    print(send_socket_control("MODE S\r\n"))
    print(send_socket_control("STRU F\r\n"))


# receive the data from the data socket then close the data socket, print data while we go
def receive_data_print(data_s):
    while 1:
        data = data_s.recv(1024)
        if not data:
            break
        data = data.decode("utf-8")
        print(data)
    data_s.close()


# receive the data for a file
def receive_data_file(path, data_s):
    with open(path, "wb") as binary_file:
        while 1:
            data = data_s.recv(1024)
            if not data:
                break
            binary_file.write(data)


# sends quit command
def quit_c():
    print(send_socket_control("QUIT\r\n"))


# parse url for password and username
def parse_pass_user(url):
    new_url = url[6:]
    if new_url[0] == "@":
        return "anonymous", ""
    else:
        ar = new_url.split(":")
        return ar[0], ar[1].split("@")[0]


# processes a mkdir on the server
def mkdir(url):
    ar = url.split("/")
    data = "MKD /" + ar[3] + "\r\n"
    print(send_socket_control(data))


# processes a mkdir on the server
def rmdir(url):
    ar = url.split("/")
    data = "RMD /" + ar[3] + "\r\n"
    print(send_socket_control(data))


# processes a DELE command on server
def dele(url):
    ar = url.split("/")
    data = "DELE /" + ar[3] + "\r\n"
    print(send_socket_control(data))


# login_init():
def login_init(arg_list):
    # parse user and pass
    parsed_user_pass = parse_pass_user(arg_list)
    # login
    login(parsed_user_pass[0], parsed_user_pass[1])


# download init
def download_init():
    # parse ip and port from server
    ip_port = pasv()
    # do download needs
    download_upload()
    # create data socket
    return create_data_socket(ip_port[0], ip_port[1])


# send data file
def send_file(filename, s):
    filesize = os.path.getsize(filename)
    with open(filename, "rb") as f:
        while True:
            bytes_read = f.read(BUFFER_SIZE)
            if not bytes_read:
                break
            s.sendall(bytes_read)
    s.close()


# receive macro for abstraction (used multiple times)
def receive_macro():
    login_init(argument_list[1])
    socket_d_1 = download_init()
    retrCommand(argument_list[1])
    # check done receiving message
    print(sock_control.recv(1024).decode("utf-8"))
    # receive data
    receive_data_file(argument_list[2], socket_d_1)


# send macro for abstraction (used multiple times)
def send_macro():
    login_init(argument_list[2])
    socket_d_2 = download_init()
    storCommand(argument_list[2])
    # send data file
    send_file(argument_list[1], socket_d_2)
    print(sock_control.recv(1024).decode("utf-8"))


# program

# command parse second
if argument_list[0] == "ls":
    login_init(argument_list[1])
    socket_d = download_init()
    # activate ls
    lsCommand(argument_list[1])
    # check done receiving message
    print(sock_control.recv(1024).decode("utf-8"))
    # receive data
    receive_data_print(socket_d)
elif argument_list[0] == "mkdir":
    login_init(argument_list[1])
    # do mkdir
    mkdir(argument_list[1])
elif argument_list[0] == "rmdir":
    login_init(argument_list[1])
    # do mkdir
    rmdir(argument_list[1])
elif argument_list[0] == "rm":
    login_init(argument_list[1])
    # do dele
    dele(argument_list[1])
elif argument_list[0] == "cp":
    # copy to this system
    if argument_list[1][:6] == "ftp://":
        receive_macro()
    # copy to the server
    else:
        send_macro()
elif argument_list[0] == "mv":
    # move to this system
    if argument_list[1][:6] == "ftp://":
        receive_macro()
        dele(argument_list[1])
    # move to the server
    else:
        send_macro()
        os.remove(argument_list[1])

quit_c()
sock_control.close()
