---
layout: post
title:  "python实现http服务器"
date:   2016-12-04 20:39:22 +0800
categories: python
---
限于水平,写的还不够好,启动后可以读取当前目录下的文件,支持html,js,css,图片
因为使用了os.fork(),windows下无法使用,可以改成threading,```t=threading.Thread(target=<func>,args=(args))
t.start()```
可以参考廖大的python教程:[TCP编程][href]

```
import os
import signal
import socket
import time

ADDRESS = (HOST, PORT) = '127.0.0.1', 8888
REQUEST_QUEUE_SIZE = 10


def request(client_conn):
    request_header=client_conn.recv(1024).decode()

    return request_header


def request_source(client_conn):
    request_header =request(client_conn)
    try:
        req_source=request_header.split()[1]
    except:
        req_source=""
    return req_source[1:]


def handle_request(client_conn):
    req_source=request_source(client_conn)
    if os.path.exists(req_source):

        if req_source.endswith('.txt') or req_source.endswith('.html'):
            header="HTTP/1.1 200 OK\nContent-type:text/html\n\n"
        elif req_source.endswith('.jpeg') or req_source.endswith(".jpg"):
            header = "HTTP/1.1 200 OK\nContent-type:image/jpeg\n\n"
        elif req_source.endswith('.gif'):
            header= "HTTP/1.1 200 OK\nContent-type:image/gif\n\n"
        elif req_source.endswith(".css"):
            header = "HTTP/1.1 200 OK\nContent-type:text/css\n\n"
        elif req_source.endswith(".js"):
            header = "HTTP/1.1 200 OK\nContent-type:text/css\n\n"
        else:
            header = "HTTP/1.1 200 OK\nContent-type:application/octet-stream\n\n"
        f = open(req_source,'rb')
        content = f.read()
        f.close()
        client_conn.sendall((header.encode() + content))
    else:
        content='HTTP/1.1 404 Not Found\n\n<h1>404 Not Found<h1>'
        client_conn.send(content.encode('utf-8'))

    time.sleep(2)

def signal_handler(signum, frame):
    pid, status = os.wait()
    print('child pid: %s stopped ,status %s'%(pid,status))


def server():
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    # setsockopt(level,optname,value)
    # 表示将SO_REUSEADDR标记为TRUE，
    # 操作系统会在服务器socket被关闭或服务器进程终止后马上释放该服务器的端口
    server_socket.bind(ADDRESS)
    server_socket.listen(REQUEST_QUEUE_SIZE)
    print('Serving HTTP on port {port} ...'.format(port=PORT))

    signal.signal(signal.SIGCHLD, signal_handler)#子进程结束后自动向父进程发送SIGCHLD信号

    while True:
        client_conn, client_address = server_socket.accept()
        pid = os.fork()
        if pid == 0:  # child
            server_socket.close()  
            handle_request(client_conn)
            client_conn.close()
            os._exit(0)
        else:
            client_conn.close()

if __name__ == '__main__':
    server()

```
少量改动,改成了threading的方式,windows下可以跑,有时候会初选UnicodeDecodeError,不知道怎么回事

```
import os
import socket
import time
import threading


ADDRESS = (HOST, PORT) = '127.0.0.1', 8888
REQUEST_QUEUE_SIZE = 10


def request(client_conn):
    request_header=client_conn.recv(1024).decode()
    return request_header


def request_source(client_conn):
    request_header =request(client_conn)
    try:
        req_source=request_header.split()[1]
    except:
        req_source=""
    return req_source[1:]

def handle_request(client_conn,addr):
    print("client %s:%s connected." % addr)
    req_source=request_source(client_conn)
    if os.path.exists(req_source):

        if req_source.endswith('.txt') or req_source.endswith('.html'):
            header="HTTP/1.1 200 OK\nContent-type:text/html\n\n"
        elif req_source.endswith('.jpeg') or req_source.endswith(".jpg"):
            header = "HTTP/1.1 200 OK\nContent-type:image/jpeg\n\n"
        elif req_source.endswith('.gif'):
            header= "HTTP/1.1 200 OK\nContent-type:image/gif\n\n"
        elif req_source.endswith(".css"):
            header = "HTTP/1.1 200 OK\nContent-type:text/css\n\n"
        elif req_source.endswith(".js"):
            header = "HTTP/1.1 200 OK\nContent-type:text/css\n\n"
        else:
            header = "HTTP/1.1 200 OK\nContent-type:application/octet-stream\n\n"
        f = open(req_source,'rb')
        content = f.read()
        f.close()
        client_conn.sendall((header.encode() + content))
    else:
        content='HTTP/1.1 404 Not Found\n\n<h1>404 Not Found<h1>'
        client_conn.send(content.encode('utf-8'))

    time.sleep(2)
    print('thread %s ended.' % threading.current_thread().name)


def server():
    print('start')
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    # setsockopt(level,optname,value)
    # 表示将SO_REUSEADDR标记为TRUE，
    # 操作系统会在服务器socket被关闭或服务器进程终止后马上释放该服务器的端口
    server_socket.bind(ADDRESS)
    server_socket.listen(REQUEST_QUEUE_SIZE)
    print('Serving HTTP on port {port} ...'.format(port=PORT))


    while True:
        client_conn, client_address = server_socket.accept()
        t=threading.Thread(target=handle_request,args=(client_conn,client_address))
        t.start()

if __name__ == '__main__':
    server()
```

[href]:http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001432004374523e495f640612f4b08975398796939ec3c000
