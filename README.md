#include <iostream>
#include <locale.h>
#include <fstream>
#include <ctime>
#include <cstdlib>
using namespace std;

const int LEVEL_COUNT = 3;
const int MAX_MOVES = 5;
const char EMPTY_SPACE = ' ';
const char WALL = '#';
const char PLAYER = 'P';
const char EXIT = 'E';
const char SPIKES = 'S';

const int MAZE_SIZE = 5;

struct Position {
    int row;
    int col;
};

void displayLevel(char maze[][MAZE_SIZE], int moveCount, const Position& playerPos, const Position& exitPos) {
    cout << "----------" << endl;
    cout << "Осталось ходов: " << moveCount << endl;

    for (int row = 0; row < MAZE_SIZE; row++) {
        for (int col = 0; col < MAZE_SIZE; col++) {
            if (row == playerPos.row && col == playerPos.col) {
                cout << PLAYER;
            }
            else if (row == exitPos.row && col == exitPos.col) {
                cout << EXIT;
            }
            else {
                cout << maze[row][col];
            }
        }
        cout << endl;
    }
}

void generateMaze(char maze[][MAZE_SIZE], Position& playerPos, Position& exitPos, int level) {
    for (int row = 0; row < MAZE_SIZE; row++) {
        for (int col = 0; col < MAZE_SIZE; col++) {
            maze[row][col] = EMPTY_SPACE;
        }
    }

    for (int i = 0; i < MAZE_SIZE * 2; i++) {
        int row = rand() % MAZE_SIZE;
        int col = rand() % MAZE_SIZE;
        maze[row][col] = WALL;
    }

    playerPos.row = rand() % MAZE_SIZE;
    playerPos.col = rand() % MAZE_SIZE;
    maze[playerPos.row][playerPos.col] = PLAYER;

    exitPos.row = rand() % MAZE_SIZE;
    exitPos.col = rand() % MAZE_SIZE;
    maze[exitPos.row][exitPos.col] = EXIT;

    // Добавляем шипы на первые два уровня
    if (level == 1 || level == 2) {
        int spikeRow = rand() % MAZE_SIZE;
        int spikeCol = rand() % MAZE_SIZE;
        maze[spikeRow][spikeCol] = SPIKES;
    }
}

void playLevel(int level, ofstream& recordFile) {
    char maze[MAZE_SIZE][MAZE_SIZE];
    Position playerPos, exitPos;
    int moveCount = MAX_MOVES;

    generateMaze(maze, playerPos, exitPos, level);

    while (true) {
        displayLevel(maze, moveCount, playerPos, exitPos);

        char move;
        cout << "Введите ваш ход (w - вверх, s - вниз, a - влево, d - вправо): ";
        cin >> move;

        int newRow = playerPos.row;
        int newCol = playerPos.col;

        if (move == 'w') {
            newRow--;
        }
        else if (move == 's') {
            newRow++;
        }
        else if (move == 'a') {
            newCol--;
        }
        else if (move == 'd') {
            newCol++;
        }
        else {
            cout << "Недопустимый ход! Попробуйте снова." << endl;
            continue;
        }

        if (newRow < 0 || newRow >= MAZE_SIZE || newCol < 0 || newCol >= MAZE_SIZE) {
            cout << "Недопустимый ход! Попробуйте снова." << endl;
            continue;
        }

        if (maze[newRow][newCol] == WALL) {
            cout << "Вы не можете пройти через стену! Попробуйте снова." << endl;
            continue;
        }

        playerPos.row = newRow;
        playerPos.col = newCol;

        moveCount--;

        // Проверка шипов
        if (level == 1 || level == 2) {
            if (maze[playerPos.row][playerPos.col] == SPIKES) {
                cout << "Вы наступили на шипы! Игра начнется заново с первого уровня." << endl;
                recordFile << "Игрок проиграл на уровне " << level << endl;
                return;
            }
        }

        // Проверка выхода из лабиринта
        if (playerPos.row == exitPos.row && playerPos.col == exitPos.col) {

            if (level == LEVEL_COUNT) {
                cout << "Поздравляем! Вы прошли все уровни лабиринта!" << endl;
                recordFile << "Игрок прошел все уровни лабиринта." << endl;
                return;
            }
            else {
                cout << "Поздравляем! Вы прошли уровень " << level << ". Переход к следующему уровню." << endl;
                recordFile << "Игрок прошел уровень " << level << endl;
                return playLevel(level + 1, recordFile);
            }
        }
    }
}

int main() {
    setlocale(LC_ALL, ""); // Установка локали для вывода сообщений на русском языке

    srand(time(NULL)); // Инициализация генератора случайных чисел

    ofstream recordFile("records.txt", ios::app);
    if (!recordFile) {
        cout << "Не удалось открыть файл records.txt" << endl;
        return 1;
    }

    recordFile << "Новая игра" << endl;

    playLevel(1, recordFile);

    recordFile << "Игра завершена" << endl;

    recordFile.close();

    return 0;
}
