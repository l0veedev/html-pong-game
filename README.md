# HTML Ping Pong Game

A lightweight, high-performance open-source Pong game built with pure HTML5 Canvas and JavaScript. Optimized for both desktop and mobile devices with smooth touch controls. Written in a single file with zero external dependencies or assets.

### How to Play
- **Desktop:** Use the **Arrow Keys** or **W, S** to control the paddle.
- **Mobile:** Touch and drag anywhere on the game canvas to smoothly move your paddle.
- Defend your side and bounce the ball back to score points against the AI.
- The ball speed increases slightly with each hit.

### Play Online
You can play the game instantly in your browser via [GitHub Pages](https://l0veedev.github.io/html-pong-game/).

---

### Source Code (`index.html`)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>HTML Ping Pong Game — Play Online | l0veedev</title>
    <meta name="description" content="A lightweight, high-performance open-source Pong game built with pure HTML5 Canvas and JavaScript. Play online against AI, mobile-friendly with smooth touch controls!">
    <meta name="keywords" content="html pong game, javascript ping pong, retro arcade, indie pet project, l0veedev">
    <meta name="author" content="l0veedev">
    <style>
        body {
            margin: 0;
            padding: 20px;
            background: #0d0d0d;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            box-sizing: border-box;
            outline: none;
            color: #fff;
            font-family: sans-serif;
            user-select: none;
            -webkit-user-select: none;
            overflow: hidden;
        }
        h1 {
            margin: 0 0 15px 0;
            font-size: 22px;
            letter-spacing: 2px;
            color: #e0e0e0;
        }
        canvas {
            background: #1a1a1a;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.8);
            border-radius: 4px;
            cursor: pointer;
            margin-bottom: 20px;
            touch-action: none;
            width: 100%;
            max-width: 600px;
            height: auto;
            aspect-ratio: 600 / 400;
        }
        .comments-container {
            width: 100%;
            max-width: 600px;
        }
    </style>
</head>
<body>
    <h1>PONG GAME</h1>
    <canvas id="board" width="600" height="400"></canvas>

    <div class="comments-container">
        <script src="[https://giscus.app/client.js](https://giscus.app/client.js)"
                data-repo="l0veedev/html-snake-game"
                data-repo-id="R_kgDOVnXJAA"
                data-category="Announcements"
                data-category-id="DIC_kwDOVnXJAM4DKrfR"
                data-mapping="pathname"
                data-strict="0"
                data-reactions-enabled="1"
                data-emit-metadata="0"
                data-input-position="bottom"
                data-theme="dark"
                data-lang="ru"
                crossorigin="anonymous"
                async>
        </script>
    </div>

    <script>
        const canvas = document.getElementById('board');
        const ctx = canvas.getContext('2d');

        const player = { x: 10, y: canvas.height / 2 - 40, width: 10, height: 80, score: 0, color: '#4CAF50' };
        const ai = { x: canvas.width - 20, y: canvas.height / 2 - 40, width: 10, height: 80, score: 0, color: '#e53935', speed: 2.8 };
        const ball = { x: canvas.width / 2, y: canvas.height / 2, radius: 8, baseSpeed: 4, speed: 4, dx: 4, dy: 2.5, color: '#ffffff' };

        const keys = {};

        window.addEventListener('keydown', e => {
            if (['ArrowUp', 'ArrowDown', 'KeyW', 'KeyS'].includes(e.code)) {
                keys[e.code] = true;
                e.preventDefault();
            }
        });

        window.addEventListener('keyup', e => {
            if (keys[e.code]) keys[e.code] = false;
        });

        const handleTouch = (e) => {
            const rect = canvas.getBoundingClientRect();
            const scaleY = canvas.height / rect.height;
            const touchY = (e.touches[0].clientY - rect.top) * scaleY;
            player.y = Math.max(0, Math.min(canvas.height - player.height, touchY - player.height / 2));
            e.preventDefault();
        };

        canvas.addEventListener('touchstart', handleTouch, { passive: false });
        canvas.addEventListener('touchmove', handleTouch, { passive: false });

        function resetBall(winner) {
            ball.x = canvas.width / 2;
            ball.y = canvas.height / 2;
            ball.speed = ball.baseSpeed;
            ball.dx = (winner === 'player' ? 1 : -1) * ball.baseSpeed;
            ball.dy = (Math.random() > 0.5 ? 1 : -1) * 2.5;
        }

        function checkCollision(b, p) {
            return b.x + b.radius > p.x && b.x - b.radius < p.x + p.width &&
                   b.y + b.radius > p.y && b.y - b.radius < p.y + p.height;
        }

        let lastTime = 0;
        const fpsInterval = 1000 / 60;

        function update() {
            if ((keys['ArrowUp'] || keys['KeyW']) && player.y > 0) player.y -= 6;
            if ((keys['ArrowDown'] || keys['KeyS']) && player.y + player.height < canvas.height) player.y += 6;

            let aiCenter = ai.y + ai.height / 2;
            if (aiCenter < ball.y - 10) ai.y += ai.speed;
            else if (aiCenter > ball.y + 10) ai.y -= ai.speed;
            ai.y = Math.max(0, Math.min(canvas.height - ai.height, ai.y));

            ball.x += ball.dx;
            ball.y += ball.dy;

            if (ball.y - ball.radius < 0 || ball.y + ball.radius > canvas.height) ball.dy = -ball.dy;

            let activePaddle = ball.x < canvas.width / 2 ? player : ai;

            if (checkCollision(ball, activePaddle)) {
                let collidePoint = (ball.y - (activePaddle.y + activePaddle.height / 2)) / (activePaddle.height / 2);
                let angleRad = (Math.PI / 4) * collidePoint;
                let direction = ball.x < canvas.width / 2 ? 1 : -1;

                ball.dx = direction * ball.speed * Math.cos(angleRad);
                ball.dy = ball.speed * Math.sin(angleRad);
                ball.speed += 0.1;
            }

            if (ball.x - ball.radius < 0) {
                ai.score++;
                resetBall('ai');
            } else if (ball.x + ball.radius > canvas.width) {
                player.score++;
                resetBall('player');
            }
        }

        function render() {
            ctx.fillStyle = '#1a1a1a';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            ctx.strokeStyle = '#333333';
            ctx.setLineDash([5, 15]);
            ctx.beginPath();
            ctx.moveTo(canvas.width / 2, 0);
            ctx.lineTo(canvas.width / 2, canvas.height);
            ctx.stroke();
            ctx.setLineDash([]);

            ctx.fillStyle = '#4CAF50';
            ctx.font = '35px sans-serif';
            ctx.fillText(player.score, canvas.width / 4, canvas.height / 5);
            ctx.fillStyle = '#e53935';
            ctx.fillText(ai.score, 3 * canvas.width / 4, canvas.height / 5);

            ctx.fillStyle = player.color;
            ctx.fillRect(player.x, player.y, player.width, player.height);
            ctx.fillStyle = ai.color;
            ctx.fillRect(ai.x, ai.y, ai.width, ai.height);

            ctx.fillStyle = ball.color;
            ctx.beginPath();
            ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2);
            ctx.fill();
        }

        function loop(timestamp) {
            if (!lastTime) lastTime = timestamp;
            let elapsed = timestamp - lastTime;

            if (elapsed >= fpsInterval) {
                lastTime = timestamp - (elapsed % fpsInterval);
                update();
                render();
            }
            requestAnimationFrame(loop);
        }

        window.focus();
        window.addEventListener('click', () => window.focus());
        window.addEventListener('touchstart', () => window.focus());
        requestAnimationFrame(loop);
    </script>
</body>
</html>
