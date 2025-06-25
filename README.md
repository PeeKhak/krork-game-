# krork-game-
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
        #controls {
            margin-bottom: 20px;
        }
        .difficulty-btn, #restart {
            padding: 10px 20px;
            font-size: 16px;
            margin: 5px;
            cursor: pointer;
            border: none;
            border-radius: 5px;
            color: white;
        }
        #hard-bot {
            background-color: #d32f2f;
        }
        #hard-bot:hover {
            background-color: #b71c1c;
        }
        #medium-bot {
            background-color: #0288d1;
        }
        #medium-bot:hover {
            background-color: #01579b;
        }
        #easy-bot {
            background-color: #4CAF50;
        }
        #easy-bot:hover {
            background-color: #388e3c;
        }
        #restart {
            background-color: #7b1fa2;
        }
        #restart:hover {
            background-color: #4a148c;
        }
    </style>
</head>
<body>
    <h1>Tic Tac Toe - 5 in a Row vs Bot</h1>
    <div id="status">Select a difficulty to start</div>
    <div id="controls">
        <button id="hard-bot" class="difficulty-btn">Hard Bot</button>
        <button id="medium-bot" class="difficulty-btn">Medium Bot</button>
        <button id="easy-bot" class="difficulty-btn">Easy Bot</button>
    </div>
    <div id="board"></div>
    <button id="restart">Restart Game</button>

    <script>
        const boardElement = document.getElementById('board');
        const statusElement = document.getElementById('status');
        const restartBtn = document.getElementById('restart');
        const hardBotBtn = document.getElementById('hard-bot');
        const mediumBotBtn = document.getElementById('medium-bot');
        const easyBotBtn = document.getElementById('easy-bot');
        const gridSize = 10;
        let gameBoard = Array(gridSize * gridSize).fill('');
        let currentPlayer = 'X'; // Human is X, Bot is O
        let gameActive = false; // Game starts inactive until difficulty is selected
        let botDifficulty = null; // No bot selected initially

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
            const index = parseInt(e.target.dataset.index);
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

        // Bot move based on difficulty
        function botMove() {
            if (!gameActive || !gameBoard.includes('')) return;
            let move = null;
            if (botDifficulty === 'hard') {
                move = findHardBotMove();
            } else if (botDifficulty === 'medium') {
                move = findMediumBotMove();
            } else if (botDifficulty === 'easy') {
                move = findEasyBotMove();
            }
            if (move !== null) {
                makeMove(move, 'O');
            }
        }

        // Hard Bot (~20% player win rate): Strong but not perfect
        function findHardBotMove() {
            // 1. Win with 5 in a row
            let move = checkForWinOrBlock('O', 5);
            if (move !== null) return move;
            // 2. Block player's 5 in a row
            move = checkForWinOrBlock('X', 5);
            if (move !== null) return move;
            // 3. Use minimax (depth 2)
            move = minimaxMove(2);
            if (move !== null) return move;
            // 4. Set up or block 4 in a row
            move = checkForWinOrBlock('O', 4);
            if (move !== null) return move;
            move = checkForWinOrBlock('X', 4);
            if (move !== null) return move;
            // 5. Set up or block 3 in a row (50% chance)
            if (Math.random() < 0.5) {
                move = checkForWinOrBlock('O', 3);
                if (move !== null) return move;
                move = checkForWinOrBlock('X', 3);
                if (move !== null) return move;
            }
            // 6. Set up or block 2 in a row (50% chance)
            if (Math.random() < 0.5) {
                move = checkForWinOrBlock('O', 2);
                if (move !== null) return move;
                move = checkForWinOrBlock('X', 2);
                if (move !== null) return move;
            }
            // 7. Scored move (clustering and center)
            return getScoredMove();
        }

        // Medium Bot (IQ ~100): Checks only for 5 in a row to win/block
        function findMediumBotMove() {
            // 1. Win with 5 in a row
            let move = checkForWinOrBlock('O', 5);
            if (move !== null) return move;
            // 2. Block player's 5 in a row
            move = checkForWinOrBlock('X', 5);
            if (move !== null) return move;
            // 3. Random move
            return getRandomMove();
        }

        // Easy Bot (IQ ~50): Mostly random, occasionally checks for 5 in a row
        function findEasyBotMove() {
            // 20% chance to check for win/block
            if (Math.random() < 0.2) {
                // 1. Win with 5 in a row
                let move = checkForWinOrBlock('O', 5);
                if (move !== null) return move;
                // 2. Block player's 5 in a row
                move = checkForWinOrBlock('X', 5);
                if (move !== null) return move;
            }
            // Random move
            return getRandomMove();
        }

        // Get random move
        function getRandomMove() {
            const emptyCells = gameBoard
                .map((val, idx) => (val === '' ? idx : null))
                .filter(val => val !== null);
            if (emptyCells.length > 0) {
                return emptyCells[Math.floor(Math.random() * emptyCells.length)];
            }
            return null;
        }

        // Minimax with alpha-beta pruning
        function minimaxMove(depthLimit) {
            let bestScore = -Infinity;
            let bestMove = null;
            const emptyCells = gameBoard
                .map((val, idx) => (val === '' ? idx : null))
                .filter(val => val !== null);

            for (const index of emptyCells) {
                gameBoard[index] = 'O';
                const score = minimax(gameBoard, 0, false, -Infinity, Infinity, depthLimit);
                gameBoard[index] = '';
                if (score > bestScore) {
                    bestScore = score;
                    bestMove = index;
                }
            }
            return bestMove;
        }

        // Minimax with simplified scoring
        function minimax(board, depth, isMaximizing, alpha, beta, depthLimit) {
            if (checkWin('O')) return 1000 - depth;
            if (checkWin('X')) return -1000 + depth;
            if (!board.includes('') || depth >= depthLimit) {
                return evaluateBoard(board);
            }

            if (isMaximizing) {
                let bestScore = -Infinity;
                const emptyCells = board
                    .map((val, idx) => (val === '' ? idx : null))
                    .filter(val => val !== null);
                for (const index of emptyCells) {
                    board[index] = 'O';
                    const score = minimax(board, depth + 1, false, alpha, beta, depthLimit);
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
                    const score = minimax(board, depth + 1, true, alpha, beta, depthLimit);
                    board[index] = '';
                    bestScore = Math.min(score, bestScore);
                    beta = Math.min(beta, bestScore);
                    if (beta <= alpha) break;
                }
                return bestScore;
            }
        }

        // Simplified board evaluation
        function evaluateBoard(board) {
            let score = 0;

            // Evaluate rows, columns, diagonals
            const lines = [];
            for (let row = 0; row < gridSize; row++) {
                for (let col = 0; col <= gridSize - 5; col++) {
                    lines.push([0, 1, 2, 3, 4].map(i => board[row * gridSize + col + i]));
                }
            }
            for (let col = 0; col < gridSize; col++) {
                for (let row = 0; row <= gridSize - 5; row++) {
                    lines.push([0, 1, 2, 3, 4].map(i => board[(row + i) * gridSize + col]));
                }
            }
            for (let row = 0; row <= gridSize - 5; row++) {
                for (let col = 0; col <= gridSize - 5; col++) {
                    lines.push([0, 1, 2, 3, 4].map(i => board[(row + i) * gridSize + col + i]));
                }
            }
            for (let row = 0; row <= gridSize - 5; row++) {
                for (let col = 4; col < gridSize; col++) {
                    lines.push([0, 1, 2, 3, 4].map(i => board[(row + i) * gridSize + col - i]));
                }
            }

            // Score lines
            for (const line of lines) {
                const oCount = line.filter(cell => cell === 'O').length;
                const xCount = line.filter(cell => cell === 'X').length;
                if (xCount === 0 && oCount >= 3) {
                    score += Math.pow(8, oCount); // Reward O's for 3+ in a row
                } else if (oCount === 0 && xCount >= 3) {
                    score -= Math.pow(8, xCount); // Penalize X's for 3+ in a row
                }
            }

            // Center control bonus
            const centerCells = [44, 45, 54, 55]; // Center 4 cells
            for (const index of centerCells) {
                if (board[index] === 'O') score += 20;
                if (board[index] === 'X') score -= 20;
            }

            return score;
        }

        // Get scored move (clustering and center preference)
        function getScoredMove() {
            const emptyCells = gameBoard
                .map((val, idx) => (val === '' ? idx : null))
                .filter(val => val !== null);
            if (emptyCells.length === 0) return null;

            let bestScore = -Infinity;
            let bestMove = null;

            for (const index of emptyCells) {
                let score = 0;
                const row = Math.floor(index / gridSize);
                const col = index % gridSize;

                // Cluster score
                const neighbors = [
                    row > 0 ? index - gridSize : null, // up
                    row < gridSize - 1 ? index + gridSize : null, // down
                    col > 0 ? index - 1 : null, // left
                    col < gridSize - 1 ? index + 1 : null, // right
                    row > 0 && col > 0 ? index - gridSize - 1 : null, // top-left
                    row > 0 && col < gridSize - 1 ? index - gridSize + 1 : null, // top-right
                    row < gridSize - 1 && col > 0 ? index + gridSize - 1 : null, // bottom-left
                    row < gridSize - 1 && col < gridSize - 1 ? index + gridSize + 1 : null, // bottom-right
                ].filter(n => n !== null && gameBoard[n] === 'O');
                score += neighbors.length * 10; // 10 points per adjacent O

                // Center preference
                const centerRow = Math.abs(row - 4.5);
                const centerCol = Math.abs(col - 4.5);
                const centerDistance = Math.sqrt(centerRow * centerRow + centerCol * centerCol);
                score += (5 - centerDistance) * 5; // Reduced bonus for center

                if (score > bestScore) {
                    bestScore = score;
                    bestMove = index;
                }
            }
            return bestMove;
        }

        // Check for potential win or block for n in a row
        function checkForWinOrBlock(player, n) {
            // Check rows
            for (let row = 0; row < gridSize; row++) {
                for (let col = 0; col <= gridSize - 5; col++) {
                    const start = row * gridSize + col;
                    const line = [0, 1, 2, 3, 4].map(i => gameBoard[start + i]);
                    if (line.filter(cell => cell === player).length === n - 1 && line.includes('')) {
                        const emptyIndex = line.indexOf('');
                        return start + emptyIndex;
                    }
                }
            }
            // Check columns
            for (let col = 0; col < gridSize; col++) {
                for (let row = 0; row <= gridSize - 5; row++) {
                    const start = row * gridSize + col;
                    const line = [0, 1, 2, 3, 4].map(i => gameBoard[start + i * gridSize]);
                    if (line.filter(cell => cell === player).length === n - 1 && line.includes('')) {
                        const emptyIndex = line.indexOf('');
                        return start + emptyIndex * gridSize;
                    }
                }
            }
            // Check main diagonals
            for (let row = 0; row <= gridSize - 5; row++) {
                for (let col = 0; col <= gridSize - 5; col++) {
                    const start = row * gridSize + col;
                    const line = [0, 1, 2, 3, 4].map(i => gameBoard[start + i * (gridSize + 1)]);
                    if (line.filter(cell => cell === player).length === n - 1 && line.includes('')) {
                        const emptyIndex = line.indexOf('');
                        return start + emptyIndex * (gridSize + 1);
                    }
                }
            }
            // Check anti-diagonals
            for (let row = 0; row <= gridSize - 5; row++) {
                for (let col = 4; col < gridSize; col++) {
                    const start = row * gridSize + col;
                    const line = [0, 1, 2, 3, 4].map(i => gameBoard[start + i * (gridSize - 1)]);
                    if (line.filter(cell => cell === player).length === n - 1 && line.includes('')) {
                        const emptyIndex = line.indexOf('');
                        return start + emptyIndex * (gridSize - 1);
                    }
                }
            }
            return null;
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

        // Start game with selected difficulty
        function startGame(difficulty) {
            botDifficulty = difficulty;
            gameBoard = Array(gridSize * gridSize).fill('');
            currentPlayer = 'X';
            gameActive = true;
            updateStatus();
            createBoard();
        }

        // Event listeners for difficulty buttons
        hardBotBtn.addEventListener('click', () => startGame('hard'));
        mediumBotBtn.addEventListener('click', () => startGame('medium'));
        easyBotBtn.addEventListener('click', () => startGame('easy'));

        // Event listener for restart button
        restartBtn.addEventListener('click', () => {
            botDifficulty = null;
            gameActive = false;
            gameBoard = Array(gridSize * gridSize).fill('');
            statusElement.textContent = 'Select a difficulty to start';
            statusElement.style.color = '#000';
            createBoard();
        });

        // Initialize the game (board is empty until difficulty is selected)
        createBoard();
    </script>
</body>
</html>
