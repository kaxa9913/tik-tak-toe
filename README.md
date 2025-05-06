
    #include <iostream>
    #include <vector>
    #include <limits>
    #include <cstdlib>
    #include <ctime>

    using namespace std;

    const char PLAYER = 'X'; // Символ игрока
    const char COMPUTER = 'O'; // Символ компьютера
    int difficulty = 1; // Уровень сложности

    class Game {
    private:
    vector<vector<char>> board; // Игровое поле

    public:
    // Конструктор: инициализация пустого поля
    Game() {
        board = vector<vector<char>>(3, vector<char>(3, ' '));
    }

    // Метод для отображения текущего состояния поля
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

    // Проверка есть ли свободные клетки
    bool isMovesLeft() {
        for (auto& row : board)
            for (auto cell : row)
                if (cell == ' ') return true;
        return false;
    }

    // Сделать ход, если клетка свободна
    bool makeMove(int row, int col, char player) {
        if (row >= 0 && row < 3 && col >= 0 && col < 3 && board[row][col] == ' ') {
            board[row][col] = player;
            return true;
        }
        return false;
    }

    // Обработка хода игрока с проверкой корректности
    void playerMove() {
        int row, col;
        while (true) {
            cout << "Введите ваш ход (строка и столбец от 0 до 2): ";
            cin >> row >> col;
            if (makeMove(row, col, PLAYER)) break;
            cout << "Неверный ход, попробуйте снова.\n";
        }
    }

    // Проверка текущего состояния поля на победителя
    char checkWinner() {
        // Проверка строк и столбцов
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

        // Проверка диагоналей
        if (board[0][0] != ' ' &&
            board[0][0] == board[1][1] &&
            board[1][1] == board[2][2])
            return board[0][0];

        if (board[0][2] != ' ' &&
            board[0][2] == board[1][1] &&
            board[1][1] == board[2][0])
            return board[0][2];

        // Нет победителя
        return ' ';
    }

    // Оценка состояния поля для алгоритма minimax
    int evaluate() {
        char winner = checkWinner();
        if (winner == COMPUTER) return +10; // Выигрыш компьютера
        else if (winner == PLAYER) return -10; // Выигрыш игрока
        else return 0; // Ничья или игра продолжается
    }

    // Реализация алгоритма minimax
    int minimax(bool isMax) {
        int score = evaluate();

        // Если есть победитель, вернуть соответствующий результат
        if (score == 10 || score == -10) return score;

        // Если ходов не осталось, ничья
        if (!isMovesLeft()) return 0;

        if (isMax) {
            int best = -1000; // Инициализация минимально возможным
            // Перебор всех пустых клеток для поиска лучшего хода
            for (int i = 0; i < 3; i++)
                for (int j = 0; j < 3; j++)
                    if (board[i][j] == ' ') {
                        board[i][j] = COMPUTER; // Сделать ход компьютера
                        best = max(best, minimax(!isMax)); // Рекурсия
                        board[i][j] = ' '; // Отменить ход
                    }
            return best;
        } else {
            int best = 1000; // Инициализация максимально возможным
            // Перебор всех пустых клеток для поиска худшего для компьютера варианта
            for (int i = 0; i < 3; i++)
                for (int j = 0; j < 3; j++)
                    if (board[i][j] == ' ') {
                        board[i][j] = PLAYER; // Сделать ход игрока
                        best = min(best, minimax(!isMax)); // Рекурсия
                        board[i][j] = ' '; // Отменить ход
                    }
            return best;
        }
    }

    // Логика выбора хода компьютера в зависимости от уровня сложности
    void computerMove() {
        int bestVal = -1000;
        int bestRow = -1, bestCol = -1;

        if (difficulty == 1) { // Легкий уровень - случайный ход
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

        else if (difficulty == 2) { // Средний уровень - блокировать или делать случайный ход
            bool moveMade = false;

            // Попытка заблокировать победу игрока
            for (int i = 0; i < 3 && !moveMade; ++i) {
                for (int j = 0; j < 3 && !moveMade; ++j) {
                    if (board[i][j] == ' ') {
                        // Проверка, если поставим сюда игрока, может выиграть
                        board[i][j] = PLAYER;
                        if (checkWinner() == PLAYER) {
                            // Блокируем
                            board[i][j] = COMPUTER;
                            moveMade = true;
                            break;
                        }
                        // Возвращение назад
                        board[i][j] = ' ';
                    }
                }
            }

            // Если не удалось блокировать, делаем случайный ход
            if (!moveMade) {
                vector<pair<int, int>> freeMoves;
                for (int i = 0; i < 3; ++i)
                    for (int j = 0; j < 3; ++j)
                        if (board[i][j] == ' ')
                            freeMoves.push_back({i, j});
                if (!freeMoves.empty()) {
                    srand(time(0));
                    auto move = freeMoves[rand() % freeMoves.size()];
                    makeMove(move.first, move.second, COMPUTER);
                }
            }
        }

        else if (difficulty == 3) { // Сложный уровень - алгоритм minimax
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
            // Сделать оптимальный ход
            makeMove(bestRow, bestCol, COMPUTER);
        }
    }

    // Основной цикл игры
    void start() {
        printBoard();
        while (true) {
            playerMove(); // Ход игрока
            printBoard();
            if (checkWinner() == PLAYER) { // Проверка победы игрока
                cout << "Вы победили!\n";
                break;
            }
            if (!isMovesLeft()) { // Если ходов не осталось, ничья
                cout << "Ничья!\n";
                break;
            }

            cout << "Ход компьютера...\n";
            computerMove(); // Ход компьютера
            printBoard();
            if (checkWinner() == COMPUTER) { // Проверка победы компьютера
                cout << "Вы проиграли.\n";
                break;
            }
            if (!isMovesLeft()) { // Если ходов не осталось, ничья
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
