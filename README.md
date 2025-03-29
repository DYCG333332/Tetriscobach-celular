<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tetris</title>
    <style>
        body { text-align: center; font-family: Arial, sans-serif; }
        canvas { background: #000; display: block; margin: auto; }
        #score { font-size: 20px; margin-top: 20px; }
        
        /* Estilos para los controles táctiles */
        .controls {
            display: flex;
            justify-content: center;
            margin-top: 20px;
        }
        .control-button {
            background-color: rgba(0, 0, 0, 0.7);
            color: white;
            padding: 20px;
            margin: 5px;
            font-size: 20px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        .control-button:active {
            background-color: rgba(0, 0, 0, 1);
        }
    </style>
</head>
<body>
    <h1>Juego de Tetris</h1>
    <canvas id="tetris" width="300" height="600"></canvas>
    <div id="score">Puntos: 0</div> <!-- Contador de puntos -->

    <!-- Controles táctiles para móvil -->
    <div class="controls">
        <button class="control-button" id="left">←</button>
        <button class="control-button" id="down">↓</button>
        <button class="control-button" id="rotate">↑</button>
        <button class="control-button" id="right">→</button>
    </div>

    <script>
        const canvas = document.getElementById("tetris");
        const context = canvas.getContext("2d");
        context.scale(30, 30);

        const tetrominos = [
            { shape: [[1, 1, 1], [0, 1, 0]], color: "red" },
            { shape: [[1, 1], [1, 1]], color: "yellow" },
            { shape: [[1, 1, 0], [0, 1, 1]], color: "green" },
            { shape: [[0, 1, 1], [1, 1, 0]], color: "blue" },
            { shape: [[1, 1, 1, 1]], color: "cyan" },
            { shape: [[1, 1, 1], [1, 0, 0]], color: "orange" },
            { shape: [[1, 1, 1], [0, 0, 1]], color: "purple" }
        ];

        let board = Array.from({ length: 20 }, () => Array(10).fill(0));
        let currentPiece = generatePiece();
        let position = { x: 4, y: 0 };
        let lastTime = 0;
        const dropInterval = 1000;
        let dropCounter = 0;
        let points = 0;

        function drawMatrix(matrix, offset) {
            matrix.forEach((row, y) => {
                row.forEach((value, x) => {
                    if (value !== 0) {
                        context.fillStyle = currentPiece.color;
                        context.fillRect(x + offset.x, y + offset.y, 1, 1);
                    }
                });
            });
        }

        function generatePiece() {
            const randomIndex = Math.floor(Math.random() * tetrominos.length);
            return { 
                shape: tetrominos[randomIndex].shape,
                color: tetrominos[randomIndex].color,
                rotation: 0
            };
        }

        function draw() {
            context.fillStyle = "black";
            context.fillRect(0, 0, canvas.width, canvas.height);
            drawMatrix(currentPiece.shape, position);
            drawMatrix(board, { x: 0, y: 0 });
            document.getElementById("score").textContent = `Puntos: ${points}`;
        }

        function movePiece(dx, dy) {
            position.x += dx;
            position.y += dy;
            if (collision()) {
                position.x -= dx;
                position.y -= dy;
                if (dy > 0) {
                    placePiece();
                }
            }
        }

        function collision() {
            return currentPiece.shape.some((row, y) => {
                return row.some((value, x) => {
                    if (value !== 0) {
                        const newX = position.x + x;
                        const newY = position.y + y;
                        return newX < 0 || newX >= 10 || newY >= 20 || board[newY][newX] !== 0;
                    }
                    return false;
                });
            });
        }

        function placePiece() {
            currentPiece.shape.forEach((row, y) => {
                row.forEach((value, x) => {
                    if (value !== 0) {
                        board[position.y + y][position.x + x] = value;
                    }
                });
            });
            clearRows();
            currentPiece = generatePiece();
            position = { x: 4, y: 0 };
            if (collision()) {
                alert("Game Over!");
                board = Array.from({ length: 20 }, () => Array(10).fill(0));
            }
        }

        function clearRows() {
            for (let row = 0; row < 20; row++) {
                if (board[row].every(cell => cell !== 0)) {
                    points += 10;
                    board.splice(row, 1);
                    board.unshift(Array(10).fill(0));
                }
            }
        }

        function rotatePiece() {
            currentPiece.rotation = (currentPiece.rotation + 1) % 4;
            currentPiece.shape = rotate(currentPiece.shape);

            if (collision()) {
                currentPiece.rotation = (currentPiece.rotation - 1 + 4) % 4;
                currentPiece.shape = rotate(currentPiece.shape);
            }
        }

        function rotate(matrix) {
            const rotated = matrix[0].map((_, index) => matrix.map(row => row[index])).reverse();
            return rotated;
        }

        function update(time = 0) {
            const deltaTime = time - lastTime;
            lastTime = time;
            dropCounter += deltaTime;

            if (dropCounter > dropInterval) {
                movePiece(0, 1);
                dropCounter = 0;
            }

            draw();
            requestAnimationFrame(update);
        }

        document.addEventListener("keydown", event => {
            if (event.key === "a") {
                movePiece(-1, 0);
            } else if (event.key === "d") {
                movePiece(1, 0);
            } else if (event.key === "s") {
                movePiece(0, 1);
            } else if (event.key === "w") {
                rotatePiece();
            }
        });

        // Controles táctiles
        document.getElementById("left").addEventListener("touchstart", () => {
            movePiece(-1, 0); // Mover a la izquierda
        });

        document.getElementById("right").addEventListener("touchstart", () => {
            movePiece(1, 0); // Mover a la derecha
        });

        document.getElementById("down").addEventListener("touchstart", () => {
            movePiece(0, 1); // Mover hacia abajo
        });

        document.getElementById("rotate").addEventListener("touchstart", () => {
            rotatePiece(); // Rotar la pieza
        });

        update();
    </script>
</body>
</html>
