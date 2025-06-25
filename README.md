# krork-game-
<!DOCTYPE html>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tic Tac Toe - 5 in a Row vs Bot (10x10 Grid)</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            background-color: #f0f0f0;
        }
        #board {
            display: grid;
            grid-template-columns: repeat(10, 60px);
            grid-gap: 3px;
            margin: 20px;
        }
        .cell {
            width: 60px;
            height: 60px;
            background: #fff;
            border: 1px solid #333;
            font-size: 24px;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
        }
        .cell:hover {
            background: #e0e0e0;
        }
        #status {
            font-size: 24px;
            margin-bottom: 20px;
            font-weight: bold;
        }
        #restart {
            padding: 10px 20px;
            font-size: 18px;
            cursor: pointer;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
        }
        #restart:hover {
            background-color: #45a049;
        }
    </style>
</head>
<body>
    <h1>Tic Tac Toe - 5 in a Row vs Bot</h1>
    <div id="status">Your turn (X)</div>
    <div id="board"></div>
    <button id="restart">Restart Game</button>

    <script>
        const boardElement = document.getElementById('board');
        const statusElement = document.getElementById('status');
        const restartBtn = document.getElementById('restart');
        const gridSize = 10;
        let gameBoard = Array(gridSize * gridSize).fill('');
        let currentPlayer = 'X'; // Human is X, Bot is O
        let gameActive = true;

        // Initialize the game board
        function createBoard() {
            boardElement.style.gridTemplateColumns = `repeat(${gridSize}, 60px)`;
            boardElement.innerHTML = '';
            gameBoard.forEach((_, index) => {
                const cell = document.createElement('div');
                cell.classList.add('cell');
                cell.dataset.index = index;
                cell.addEventListener('click', handleCellClick);
                boardElement.appendChild(cell);
            });
        }

        // Handle human cell click
        function handleCellClick(e) {
            if (currentPlayer !== 'X' || !gameActive) return; // Only allow human (X) to click
            const index = e.target.dataset.index;
            if (gameBoard[index] === '') {
                makeMove(index, 'X');
                if (gameActive && gameBoard.includes('')) {
                    setTimeout(botMove, 500); // Bot moves after a short delay
                }
            }
        }

        // Make a move (for both human and bot)
        function makeMove(index, player) {
            gameBoard[index] = player;
            boardElement.children[index].textContent = player;
            updateStatus();
            if (checkWin(player)) {
                statusElement.textContent = `${player === 'X' ? 'You win! 5 in a row!' : 'Bot wins! 5 in a row!'}`;
                statusElement.style.color = '#4CAF50';
                gameActive = false;
            } else if (!gameBoard.includes('')) {
                statusElement.textContent = "It's a draw!";
                statusElement.style.color = '#FF5733';
                gameActive = false;
            } else {
                currentPlayer = player === 'X' ? 'O' : 'X';
                updateStatus();
            }
        }

        // Bot move with IQ 200 (minimax algorithm)
        function botMove() {
            if (!gameActive || !gameBoard.includes('')) return;
            const bestMove = findBestMove();
            if (bestMove !== null) {
                makeMove(bestMove, 'O');
            }
        }

        // Minimax algorithm for best move
        function findBestMove() {
            let bestScore = -Infinity;
            let bestMove = null;
            const emptyCells = gameBoard
                .map((val, idx) => (val === '' ? idx : null))
                .filter(val => val !== null);

            for (const index of emptyCells) {
                gameBoard[index] = 'O';
                const score = minimax(gameBoard, 0, false, -Infinity, Infinity);
                gameBoard[index] = '';
                if (score > bestScore) {
                    bestScore = score;
                    bestMove = index;
                }
            }
            return bestMove;
        }

        // Minimax algorithm with alpha-beta pruning
        function minimax(board, depth, isMaximizing, alpha, beta) {
            if (checkWin('O')) return 100 - depth;
            if (checkWin('X')) return -100 + depth;
            if (!board.includes('')) return 0;

            if (isMaximizing) {
                let bestScore = -Infinity;
                const emptyCells = board
                    .map((val, idx) => (val === '' ? idx : null))
                    .filter(val => val !== null);
                for (const index of emptyCells) {
                    board[index] = 'O';
                    const score = minimax(board, depth + 1, false, alpha, beta);
                    board[index] = '';
                    bestScore = Math.max(score, bestScore);
                    alpha = Math.max(alpha, bestScore);
                    if (beta <= alpha) break;
                }
                return bestScore;
            } else {
                let bestScore = Infinity;
                const emptyCells = board
                    .map((val, idx) => (val === '' ? idx : null))
                    .filter(val => val !== null);
                for (const index of emptyCells) {
                    board[index] = 'X';
                    const score = minimax(board, depth + 1, true, alpha, beta);
                    board[index] = '';
                    bestScore = Math.min(score, bestScore);
                    beta = Math.min(beta, bestScore);
                    if (beta <= alpha) break;
                }
                return bestScore;
            }
        }

        // Update status message
        function updateStatus() {
            if (!gameActive) return;
            statusElement.textContent = currentPlayer === 'X' 
                ? `Your turn (X)`
                : `Bot's turn (O)`;
            statusElement.style.color = '#000';
        }

        // Check for a win (5 in a row)
        function checkWin(player) {
            // Check rows
            for (let row = 0; row < gridSize; row++) {
                for (let col = 0; col <= gridSize - 5; col++) {
                    const start = row * gridSize + col;
                    if ([0, 1, 2, 3, 4].every(i => gameBoard[start + i] === player)) {
                        return true;
                    }
                }
            }
            // Check columns
            for (let col = 0; col < gridSize; col++) {
                for (let row = 0; row <= gridSize - 5; row++) {
                    const start = row * gridSize + col;
                    if ([0, 1, 2, 3, 4].every(i => gameBoard[start + i * gridSize] === player)) {
                        return true;
                    }
                }
            }
            // Check main diagonals
            for (let row = 0; row <= gridSize - 5; row++) {
                for (let col = 0; col <= gridSize - 5; col++) {
                    const start = row * gridSize + col;
                    if ([0, 1, 2, 3, 4].every(i => gameBoard[start + i * (gridSize + 1)] === player)) {
                        return true;
                    }
                }
            }
            // Check anti-diagonals
            for (let row = 0; row <= gridSize - 5; row++) {
                for (let col = 4; col < gridSize; col++) {
                    const start = row * gridSize + col;
                    if ([0, 1, 2, 3, 4].every(i => gameBoard[start + i * (gridSize - 1)] === player)) {
                        return true;
                    }
                }
            }
            return false;
        }

        // Restart the game
        function restartGame() {
            gameBoard = Array(gridSize * gridSize).fill('');
            currentPlayer = 'X';
            gameActive = true;
            updateStatus();
            createBoard();
        }

        // Event listener for restart button
        restartBtn.addEventListener('click', restartGame);

        // Initialize the game
        createBoard();
    </script>
</body>
</html>
</html>
        socket.on('error', (message) => {
            alert(message);
        });
    </script>
</body>
</html>
