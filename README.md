# programacionctpsanisidro
#include <iostream>
#include <fstream>
#include <vector>
#include <pthread.h>
using namespace std;

struct ThreadArgs {
    int start;
    int end;
    string filename;
};

// Función para verificar si un número es primo
bool esPrimo(int num) {
    if (num <= 1) return false;
    for (int i = 2; i * i <= num; i++) {
        if (num % i == 0) return false;
    }
    return true;
}

// Función para obtener las combinaciones de dos números primos que sumen el número par dado
vector<pair<int, int>> obtenerSumaPrimos(int num) {
    vector<pair<int, int>> sumas;
    for (int i = 2; i <= num / 2; i++) {
        if (esPrimo(i) && esPrimo(num - i)) {
            sumas.push_back({i, num - i});
        }
    }
    return sumas;
}

// Función que se ejecuta en cada hilo para generar los resultados y escribirlos en el archivo
void* generarGoldbach(void* arg) {
    ThreadArgs* args = static_cast<ThreadArgs*>(arg);

    ofstream archivo(args->filename, ios_base::app);
    if (!archivo.is_open()) {
        cerr << "Error al abrir el archivo: " << args->filename << endl;
        return nullptr;
    }

    for (int num = args->start; num <= args->end; num += 2) {
        vector<pair<int, int>> sumasPrimos = obtenerSumaPrimos(num);
        if (!sumasPrimos.empty()) {
            archivo << "Numero par: " << num << endl;
            for (const auto& suma : sumasPrimos) {
                archivo << suma.first << " + " << suma.second << " = " << num << endl;
            }
            archivo << "---------------------------" << endl;
        }
    }

    archivo.close();
    pthread_exit(nullptr);
}

int main(int argc, char* argv[]) {
    if (argc != 3) {
        cout << "Uso: " << argv[0] << " <limite> <nombre_archivo>" << endl;
        return 1;
    }

    int limite = stoi(argv[1]);
    string nombreArchivo = argv[2];

    pthread_t thread;
    ThreadArgs threadArgs;
    threadArgs.start = 4; // Iniciamos en 4 porque la conjetura de Goldbach aplica solo a números pares mayores que 2
    threadArgs.end = limite;
    threadArgs.filename = nombreArchivo;

    int result = pthread_create(&thread, nullptr, generarGoldbach, &threadArgs);
    if (result != 0) {
        cerr << "Error al crear el hilo." << endl;
        return 1;
    }

    // Esperar a que el hilo termine
    pthread_join(thread, nullptr);

    cout << "Se han generado los resultados en el archivo " << nombreArchivo << endl;

    return 0;
}
