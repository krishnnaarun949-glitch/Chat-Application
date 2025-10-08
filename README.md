input:

#include <iostream>
#include <cstring>
#include <unistd.h>
#include <arpa/inet.h>

using namespace std;

#define PORT 8080
#define BUFFER_SIZE 1024

int main() {
    int server_fd, new_socket;
    struct sockaddr_in address;
    int addrlen = sizeof(address);
    char buffer[BUFFER_SIZE] = {0};

    // Create socket
    if ((server_fd = socket(AF_INET, SOCK_STREAM, 0)) == 0) {
        perror("Socket failed");
        return 1;
    }

    // Set address
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);

    // Bind
    if (bind(server_fd, (struct sockaddr*)&address, sizeof(address)) < 0) {
        perror("Bind failed");
        return 1;
    }

    // Listen
    if (listen(server_fd, 3) < 0) {
        perror("Listen failed");
        return 1;
    }

    cout << "Server started. Waiting for connection on port " << PORT << "...\n";

    // Accept a client
    if ((new_socket = accept(server_fd, (struct sockaddr*)&address, (socklen_t*)&addrlen)) < 0) {
        perror("Accept failed");
        return 1;
    }

    cout << "Client connected.\nType 'exit' to quit.\n";

    // Chat loop
    while (true) {
        memset(buffer, 0, BUFFER_SIZE);
        int valread = read(new_socket, buffer, BUFFER_SIZE);
        if (valread <= 0) {
            cout << "Client disconnected.\n";
            break;
        }

        if (strcmp(buffer, "exit") == 0) {
            cout << "Client exited the chat.\n";
            break;
        }

        cout << "Client: " << buffer << endl;

        cout << "You: ";
        string reply;
        getline(cin, reply);
        send(new_socket, reply.c_str(), reply.size(), 0);

        if (reply == "exit") {
            cout << "Exiting chat.\n";
            break;
        }
    }

    close(new_socket);
    close(server_fd);
    return 0;
}


output:

Server started. Waiting for connection on port 8080...
