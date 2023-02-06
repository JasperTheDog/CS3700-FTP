Design:
My approach was to use Python to make a simple 1 to 1 algorithm to do exactly what I needed for each command. I took in the arguments from the command line and parsed to 
represent each command and its arguments. I made sure to abstract similar functionalites across commands. For instance all the data socket commands needed certain setup to begin
data transfer, this included MODE TYPE and STRU commands to the FTP server. I abstracted this to a single function and called it as needed. I did the same thing for cp and mv command line
arguments. They are very similar to eachother, with the only difference being the added deletion of the file from the sender. I simply abstracted the similar parts to a function and called it
in both scenarios. This was my general approach to the design between command line arguments. In general I also abstracted a send_control_socket data for all commands to the FTP server. This 
made it very easy to call on the server for any FTP command.

Challenges: 
The biggest challenge I had was reading and sending data over the data socket. I had to read the data from the data socket and write it to a file. This took a lot of research into how to convert 
files into bytes and from bytes, since this is how the socket sends them. Also the socket sent them in chuncks, so I had to read the data in chunks and write it to the file. I had a lot of errors doing this
and it took many different iterations to figure out how the socket and files were communicating properly. I also had to make sure to close the data socket after each command, otherwise the server would
not know when the data transfer was over.

Testing:
I made sure to print all the feedback from the server over the control socket to the console. This helped with testing immensely because I could see what the server was doing and what it was expecting
and where my program was going wrong. Almost always when I had a simple logical error or off by one, it would result in the program never exiting, making it obvious a infinite while loop was occuring or more 
importantly a SOCKET was LISTEN AND NOT RECEIVING. This was a huge help in debugging, once I realized that. Whenever I saw that I made sure to look for where I was unecessarily listening for data either on the 
control or data socket needlessly. I also made sure to test all the commands and their arguments. I made sure to test all the edge cases and make sure they were handled properly. I did this through simplly testing
the ftp on my commandline. I found through this that my program was not handling the case of subdirectories for every single command. This was because when I parsed the command line arguments I turned the argument
ftp//:......../path I would split this string using .split("/") giving me and array with [ftp,,.....,path], but I did not account for the fact that the path could be a subdirectory with multiple /. To fix this I limited the split to only 3 splits by calling .split("/",3).