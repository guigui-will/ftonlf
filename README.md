#include <winsock2.h>
#include <ws2tcpip.h>
#include <iostream>
#include <string>

#pragma comment(lib, "ws2_32.lib")

#define PORT 8080
#define PASSWORD "12345"

int main() {
    WSADATA wsaData;
    SOCKET serverSocket, clientSocket;
    sockaddr_in serverAddr, clientAddr;
    int clientAddrSize = sizeof(clientAddr);
    char buffer[1024] = {0};

    // Inicializar Winsock
    if (WSAStartup(MAKEWORD(2,2), &wsaData) != 0) {
        std::cerr << "Erro ao inicializar Winsock\n";
        return 1;
    }

    // Criar socket
    serverSocket = socket(AF_INET, SOCK_STREAM, 0);
    if (serverSocket == INVALID_SOCKET) {
        std::cerr << "Erro ao criar socket\n";
        WSACleanup();
        return 1;
    }

    // Configurar endereço
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_addr.s_addr = INADDR_ANY;
    serverAddr.sin_port = htons(PORT);

    // Bind
    if (bind(serverSocket, (sockaddr*)&serverAddr, sizeof(serverAddr)) == SOCKET_ERROR) {
        std::cerr << "Erro no bind\n";
        closesocket(serverSocket);
        WSACleanup();
        return 1;
    }

    // Listen
    listen(serverSocket, SOMAXCONN);
    std::cout << "Servidor aguardando conexão...\n";

    // Aceitar conexão
    clientSocket = accept(serverSocket, (sockaddr*)&clientAddr, &clientAddrSize);
    if (clientSocket == INVALID_SOCKET) {
        std::cerr << "Erro no accept\n";
        closesocket(serverSocket);
        WSACleanup();
        return 1;
    }

    // Receber senha
    recv(clientSocket, buffer, sizeof(buffer), 0);
    if (strcmp(buffer, PASSWORD) == 0) {
        std::cout << "Senha correta! Conexão autorizada.\n";
        std::string msg = "Dados secretos: Transferência concluída!";
        send(clientSocket, msg.c_str(), msg.size(), 0);
    } else {
        std::cout << "Senha incorreta. Conexão recusada.\n";
        std::string msg = "Acesso negado!";
        send(clientSocket, msg.c_str(), msg.size(), 0);
    }

    // Fechar sockets
    closesocket(clientSocket);
    closesocket(serverSocket);
    WSACleanup();
    return 0;
}
