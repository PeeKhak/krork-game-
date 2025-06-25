# krork-game-
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tic Tac Toe - 5 in a Row Multiplayer (10x10 Grid)</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            background-color: #f0f0f0;
        }
        #lobby, #game {
            text-align: center;
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
        #room-id, #join-room-id {
            margin: 10px;
            padding: 5px;
            font-size: 16px;
        }
        #create-room, #join-room {
            padding: 10px 20px;
            font-size: 16px;
            cursor: pointer;
            background-color: #2196F3;
            color: white;
            border: none;
            border-radius: 5px;
        }
        #create-room:hover, #join-room:hover {
            background-color: #1976D2;
        }
    </style>
</head>
<body>
    <h1>Tic Tac Toe - 5 in a Row Multiplayer</h1>
    <div id="lobby">
        <h2>Create or Join a Game Room</h2>
        <div>
            <button id="create-room">Create Room</button>
        </div>
        <div>
            <input id="join-room-id" type="text" placeholder="Enter Room ID">
            <button id="join-room">Join Room</button>
        </div>
    </div>
    <div id="game" style="display: none;">
        <div id="status">Waiting for opponent...</div>
        <div id="board"></div>
        <button id="restart">Restart Game</button>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/4.1.2/socket.io.min.js"></script>
    <script>
        const socket = io();
        const lobby = document.getElementById('lobby');
        const game = document.getElementById('game');
        const board = document.getElementById('board');
        const status = document.getElementById('status');
        const restartBtn = document.getElementById('restart');
        const createRoomBtn = document.getElementById('create-room');
        const joinRoomBtn = document.getElementById('join-room');
        const joinRoomIdInput = document.getElementById('join-room-id');
        const gridSize = 10;
        let gameBoard = Array(gridSize * gridSize).fill('');
        let playerSymbol = null;
        let currentPlayer = 'X';
        let playsLeft = { X: 5, O: 5 };
        let gameActive = false;
        let roomId = null;

        // Initialize the game board
        function createBoard() {
            board.style.gridTemplateColumns = `repeat(${gridSize}, 60px)`;
            board.innerHTML = '';
            gameBoard.forEach((_, index) => {
                const cell = document.createElement('div');
                cell.classList.add('cell');
                cell.dataset.index = index;
                cell.addEventListener('click', handleCellClick);
                board.appendChild(cell);
            });
        }

        // Handle cell click
        function handleCellClick(e) {
            if (!gameActive || playerSymbol !== currentPlayer) return;
            const index = e.target.dataset.index;
            if (gameBoard[index] === '' && playsLeft[playerSymbol] > 0) {
                socket.emit('makeMove', { roomId, index, player: playerSymbol });
            }
        }

        // Update board with move
        function makeMove(index, player) {
            gameBoard[index] = player;
            board.children[index].textContent = player;
            playsLeft[player]--;
            updateStatus();
            if (checkWin(player)) {
                status.textContent = `${player === playerSymbol ? 'You win! 5 in a row!' : 'Opponent wins! 5 in a row!'}`;
                status.style.color = '#4CAF50';
                gameActive = false;
            } else if (playsLeft.X === 0 && playsLeft.O === 0) {
                status.textContent = "It's a draw!";
                status.style.color = '#FF5733';
                gameActive = false;
            } else {
                currentPlayer = player === 'X' ? 'O' : 'X';
                updateStatus();
            }
        }

        // Update status message
        function updateStatus() {
            if (!gameActive) return;
            status.textContent = currentPlayer === playerSymbol 
                ? `Your turn (${playerSymbol}) - Plays left: ${playsLeft[playerSymbol]}`
                : `Opponent's turn - Plays left: ${playsLeft[currentPlayer]}`;
            status.style.color = '#000';
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
            // Check main diagonals (top-left to bottom-right)
            for (let row = 0; row <= gridSize - 5; row++) {
                for (let col = 0; col <= gridSize - 5; col++) {
                    const start = row * gridSize + col;
                    if ([0, 1, 2, 3, 4].every(i => gameBoard[start + i * (gridSize + 1)] === player)) {
                        return true;
                    }
                }
            }
            // Check anti-diagonals (top-right to bottom-left)
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

        // Create room
        createRoomBtn.addEventListener('click', () => {
            socket.emit('createRoom');
        });

        // Join room
        joinRoomBtn.addEventListener('click', () => {
            const inputRoomId = joinRoomIdInput.value.trim();
            if (inputRoomId) {
                socket.emit('joinRoom', inputRoomId);
            } else {
                alert('Please enter a Room ID');
            }
        });

        // Restart game
        restartBtn.addEventListener('click', () => {
            socket.emit('restartGame', roomId);
        });

        // Socket.IO event handlers
        socket.on('roomCreated', (id) => {
            roomId = id;
            playerSymbol = 'X';
            lobby.style.display = 'none';
            game.style.display = 'block';
            status.textContent = `Room ID: ${roomId} - Waiting for opponent...`;
            createBoard();
        });

        socket.on('roomJoined', (data) => {
            roomId = data.roomId;
            playerSymbol = data.playerSymbol;
            lobby.style.display = 'none';
            game.style.display = 'block';
            gameActive = true;
            playsLeft = { X: 5, O: 5 };
            currentPlayer = 'X';
            updateStatus();
            createBoard();
        });

        socket.on('moveMade', (data) => {
            makeMove(data.index, data.player);
        });

        socket.on('gameRestarted', () => {
            gameBoard = Array(gridSize * gridSize).fill('');
            playsLeft = { X: 5, O: 5 };
            currentPlayer = 'X';
            gameActive = true;
            updateStatus();
            createBoard();
        });

        socket.on('error', (message) => {
            alert(message);
        });
    </script>
</body>
</html>
