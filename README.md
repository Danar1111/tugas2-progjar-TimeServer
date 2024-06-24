# tugas2-progjar-TimeServer

```py
from socket import socket, AF_INET, SOCK_STREAM, SOL_SOCKET, SO_REUSEADDR
import threading
import logging
from time import gmtime, strftime

class TimeService:
    def __init__(self):
        self.response_enc = "utf-8"

    def send_time(self, conn):
        time_str = strftime('%H:%M:%S', gmtime())
        response = f"JAM {time_str}\r\n"
        conn.sendall(response.encode(self.response_enc))

    def close_connection(self, conn):
        response = "Connection successfully terminated\r\n"
        conn.sendall(response.encode(self.response_enc))
        conn.close()

    def error_response(self, conn):
        response = "Invalid command received\r\n"
        conn.sendall(response.encode(self.response_enc))

class ClientManager(threading.Thread):
    def __init__(self, conn, addr):
        super().__init__()
        self.conn = conn
        self.addr = addr
        self.handler = TimeService()

    def process_requests(self):
        while True:
            data = self.conn.recv(32)
            if data:
                command = data.decode('utf-8').strip()
                if command == 'TIME':
                    self.handler.send_time(self.conn)
                elif command == 'QUIT':
                    self.handler.close_connection(self.conn)
                    break
                else:
                    self.handler.error_response(self.conn)
            else:
                break

    def run(self):
        self.process_requests()
        self.conn.close()

class NetworkServer(threading.Thread):
    def __init__(self, host, port):
        super().__init__()
        self.host = host
        self.port = port
        self.active_connections = []
        self.server_socket = socket(AF_INET, SOCK_STREAM)
        self.server_socket.setsockopt(SOL_SOCKET, SO_REUSEADDR, 1)

    def activate_server(self):
        self.server_socket.bind((self.host, self.port))
        self.server_socket.listen(5)
        logging.info(f"Server active on {self.host}:{self.port}")

        while True:
            client_conn, client_addr = self.server_socket.accept()
            logging.info(f"New client connected: {client_addr}")
            new_client = ClientManager(client_conn, client_addr)
            new_client.start()
            self.active_connections.append(new_client)

    def run(self):
        self.activate_server()

def main():
    server = NetworkServer('0.0.0.0', 45000)
    server.start()

if __name__ == "__main__":
    logging.basicConfig(level=logging.INFO)
    main()
```
