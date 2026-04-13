##

As I discussed in my show users command, the first entry on the output was a serial connection from the location 0.0.0.0. This was an odd output because as I discussed the command is only supposed to show the active connections, and I was accessing the switch via SSH. So I ran a variation of the show user command, which was the show user login-history command, which provided a log of the successful logins into the switch, specifically when, who, what and where of each login. Here are the result of the command:
```
|       Login Time      |     Username     |    Protocol  |     Location    | 
|---------------------- | ---------------- | ------------ | ----------------|
| 27-Oct-2006 13:47:20  |    admin         |      SSH     |   192.168.1.20  |
| 27-Oct-2006 13:31:40  |    admin         |      SSH     |   192.168.1.20  |
| 27-Oct-2006 12:43:01  |    admin         |      SSH     |   192.168.1.20  |
| 27-Oct-2006 11:43:10  |    admin         |      SSH     |   192.168.1.20  |
| 26-Oct-2006 13:02:49  |    admin         |      SSH     |   192.168.1.20  |
| 26-Oct-2006 12:49:09  |    admin         |      SSH     |   192.168.1.20  |
| 26-Oct-2006 12:40:17  |    admin         |      SSH     |   192.168.1.20  |
| 26-Oct-2006 12:37:11  |    admin         |      SSH     |   192.168.1.20  |
| 26-Oct-2006 11:59:34  |    admin         |      SSH     |   192.168.1.20  |
| 25-Oct-2006 15:13:02  |    admin         |      SSH     |   192.168.1.20  |
```
From these results we can see that every single login attempt has been via SSH and from my ip address. So the first result of the show users output most likely is a ghost entry and is the result of me not properly terminating my connection after using the console cable.
