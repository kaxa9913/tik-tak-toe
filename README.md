# tik-tak-toe
#include <iostream>
#include <vector>
#include <limits>
#include <cstdlib>
#include <ctime>

using namespace std;

const char PLAYER = 'X';
const char COMPUTER = 'O';
int difficulty = 1;

class Game {
private:
    vector<vector<char>> board;

public:
    Game() {
        board = vector<vector<char>>(3, vector<char>(3, ' '));
    }

    void printBoard() {
        cout << "-------------\n";
        for (int i = 0; i < 3; i++) {
            cout << "| ";
            for (int j = 0; j < 3; j++) {
                cout << board[i][j] << " | ";
            }
            cout << "\n-------------\n";
        }
    }

    bool isMovesLeft() {
        for (auto& row : board)
            for (auto cell : row)
                if (cell == ' ') return true;
        return false;
    }

    bool makeMove(int row, int col, char player) {
        if (board[row][col] == ' ') {
            board[row][col] = player;
            return true;
        }
        return false;
    }

    void playerMove() {
        int row, col;
        while (true) {
            cout << "Введите ваш ход (строка и столбец от 0 до 2): ";
            cin >> row >> col;
            if (makeMove(row, col, PLAYER)) break;
            cout << "Неверный ход, попробуйте снова.\n";
        }
    }

    char checkWinner() {
        // Проверка по строкам, столбцам и диагоналям
        for (int i = 0; i < 3; i++) {
            if (board[i][0] != ' ' &&
                board[i][0] == board[i][1] &&
                board[i][1] == board[i][2])
                return board[i][0];

            if (board[0][i] != ' ' &&
                board[0][i] == board[1][i] &&
                board[1][i] == board[2][i])
                return board[0][i];
        }

        if (board[0][0] != ' ' &&
            board[0][0] == board[1][1] &&
            board[1][1] == board[2][2])
            return board[0][0];

        if (board[0][2] != ' ' &&
            board[0][2] == board[1][1] &&
            board[1][1] == board[2][0])
            return board[0][2];

        return ' ';
    }

    int evaluate() {
        char winner = checkWinner();
        if (winner == COMPUTER) return +10;
        else if (winner == PLAYER) return -10;
        else return 0;
    }

    int minimax(bool isMax) {
        int score = evaluate();
        if (score == 10 || score == -10) return score;
        if (!isMovesLeft()) return 0;

        if (isMax) {
            int best = -1000;
            for (int i = 0; i < 3; i++)
                for (int j = 0; j < 3; j++)
                    if (board[i][j] == ' ') {
                        board[i][j] = COMPUTER;
                        best = max(best, minimax(!isMax));
                        board[i][j] = ' ';
                    }
            return best;
        } else {
            int best = 1000;
            for (int i = 0; i < 3; i++)
                for (int j = 0; j < 3; j++)
                    if (board[i][j] == ' ') {
                        board[i][j] = PLAYER;
                        best = min(best, minimax(!isMax));
                        board[i][j] = ' ';
                    }
            return best;
        }
    }

    void computerMove() {
        int bestVal = -1000;
        int bestRow = -1, bestCol = -1;

        if (difficulty == 1) { // Easy: random
            vector<pair<int, int>> moves;
            for (int i = 0; i < 3; ++i)
                for (int j = 0; j < 3; ++j)
                    if (board[i][j] == ' ')
                        moves.push_back({i, j});
            if (!moves.empty()) {
                srand(time(0));
                auto move = moves[rand() % moves.size()];
                makeMove(move.first, move.second, COMPUTER);
            }
        }

        else if (difficulty == 2) { // Medium: block or random
            for (int i = 0; i < 3; ++i)
                for (int j = 0; j < 3; ++j)
                    if (board[i][j] == ' ') {
                        board[i][j] = PLAYER;
                        if (checkWinner() == PLAYER) {
                            board[i][j] = COMPUTER;
                            return;
                        }
                        board[i][j] = ' ';
                    }

            computerMove(); // fallback to random
        }

        else if (difficulty == 3) { // Hard: Minimax
            for (int i = 0; i < 3; i++)
                for (int j = 0; j < 3; j++)
                    if (board[i][j] == ' ') {
                        board[i][j] = COMPUTER;
                        int moveVal = minimax(false);
                        board[i][j] = ' ';
                        if (moveVal > bestVal) {
                            bestRow = i;
                            bestCol = j;
                            bestVal = moveVal;
                        }
                    }
            makeMove(bestRow, bestCol, COMPUTER);
        }
    }

    void start() {
        printBoard();
        while (true) {
            playerMove();
            printBoard();
            if (checkWinner() == PLAYER) {
                cout << "Вы победили!\n";
                break;
            }
            if (!isMovesLeft()) {
                cout << "Ничья!\n";
                break;
            }

            cout << "Ход компьютера...\n";
            computerMove();
            printBoard();
            if (checkWinner() == COMPUTER) {
                cout << "Вы проиграли.\n";
                break;
            }
            if (!isMovesLeft()) {
                cout << "Ничья!\n";
                break;
            }
        }
    }
};

int main() {
    cout << "=== КРЕСТИКИ-НОЛИКИ ===\n";
    cout << "Выберите уровень сложности:\n";
    cout << "1 - Лёгкий\n2 - Средний\n3 - Сложный\n";
    cin >> difficulty;

    Game game;
    game.start();

    return 0;
}
