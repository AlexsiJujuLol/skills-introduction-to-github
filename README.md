<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Advanced UI with Discord Invite</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: black;
            font-family: Arial, sans-serif;
            overflow: hidden;
        }

        .container {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: white;
        }

        h1 {
            font-size: 3rem;
            text-shadow: 0 0 10px #ffffff, 0 0 20px #ff00ff, 0 0 30px #ff00ff, 0 0 40px #ff00ff;
            margin-bottom: 20px;
        }

        .discord-invite {
            padding: 20px;
            background-color: #7289da;
            color: white;
            font-size: 1.5rem;
            border: none;
            border-radius: 10px;
            cursor: pointer;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
            transition: all 0.3s ease;
        }

        .discord-invite:hover {
            background-color: #5b6e98;
            box-shadow: 0 0 15px rgba(0, 0, 0, 0.7);
        }

        .glowing-dots {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
        }

        .dot {
            position: absolute;
            border-radius: 50%;
            background-color: rgba(255, 255, 255, 0.8);
            animation: glowing 5s infinite alternate;
        }

        @keyframes glowing {
            0% {
                transform: scale(0.5);
                opacity: 0.5;
            }
            100% {
                transform: scale(1.5);
                opacity: 1;
            }
        }
    </style>
</head>
<body>

    <div class="container">
        <h1>Join Our Discord Server!</h1>
        <button class="discord-invite" onclick="window.location.href='https://discord.gg/XCfbkJt5'">
            Join Now
        </button>
    </div>

    <div class="glowing-dots" id="dots-container"></div>

    <script>
        // Generate glowing dots on the screen
        const numberOfDots = 100;
        const dotsContainer = document.getElementById('dots-container');

        for (let i = 0; i < numberOfDots; i++) {
            const dot = document.createElement('div');
            dot.classList.add('dot');
            dot.style.width = `${Math.random() * 10 + 5}px`;  // Random size
            dot.style.height = dot.style.width;
            dot.style.top = `${Math.random() * 100}vh`;
            dot.style.left = `${Math.random() * 100}vw`;

            // Randomize animation duration and delay
            dot.style.animationDuration = `${Math.random() * 3 + 3}s`;
            dot.style.animationDelay = `${Math.random() * 2}s`;

            dotsContainer.appendChild(dot);
        }
    </script>

</body>
</html>
