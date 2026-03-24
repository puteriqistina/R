<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Phonics Drive: 10 Level Challenge</title>
    <style>
        :root {
            --road-color: #444;
            --grass-color: #27ae60;
            --sky-color: #87CEEB;
        }

        body {
            margin: 0; padding: 0;
            font-family: 'Arial', sans-serif;
            background-color: var(--sky-color);
            text-align: center;
            overflow: hidden;
        }

        #header { background: white; padding: 15px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); }

        /* SCROLLING GAME WORLD */
        #game-container {
            position: relative;
            height: 300px;
            background-color: var(--grass-color);
            margin-top: 20px;
            border-top: 10px solid #2ecc71;
            overflow: hidden; /* Keeps car inside */
        }

        #road {
            position: absolute;
            bottom: 50px;
            width: 200%; /* Double width for scrolling effect */
            height: 100px;
            background-color: var(--road-color);
            border-top: 4px dashed white;
            border-bottom: 4px dashed white;
            left: 0;
            transition: left 1.5s ease-in-out;
        }

        #car {
            position: absolute;
            bottom: 60px;
            left: 100px; /* Car stays around here */
            font-size: 60px;
            z-index: 10;
            transform: scaleX(-1); /* Faces Right */
            animation: driveVibration 0.2s infinite;
        }

        @keyframes driveVibration {
            0% { transform: scaleX(-1) translateY(0px); }
            50% { transform: scaleX(-1) translateY(-2px); }
            100% { transform: scaleX(-1) translateY(0px); }
        }

        .obstacle {
            position: absolute;
            bottom: 65px;
            font-size: 50px;
            left: 400px; /* Positioned in front of car */
            transition: transform 0.5s, opacity 0.5s;
        }

        #sentence-box {
            background: white;
            display: inline-block;
            padding: 20px 40px;
            border-radius: 50px;
            font-size: 24px;
            margin: 20px;
            border: 4px solid #3498db;
            min-width: 400px;
            box-shadow: 0 5px 15px rgba(0,0,0,0.1);
        }

        .highlight { color: #e74c3c; font-weight: bold; text-decoration: underline; }

        button {
            padding: 15px 40px; font-size: 18px; cursor: pointer;
            border-radius: 50px; border: none; background: #e67e22;
            color: white; box-shadow: 0 5px #d35400; font-weight: bold;
        }

        button:active { transform: translateY(3px); box-shadow: 0 2px #d35400; }
        
        #feedback { font-size: 20px; margin-top: 15px; font-weight: bold; height: 30px; }
        .success { color: #27ae60; }
        .error { color: #e74c3c; }
    </style>
</head>
<body>

    <div id="header">
        <h1>🏙️ Phonics City: 10-Level Drive</h1>
        <p>Read clearly to pass the obstacles!</p>
        <h3>Score: <span id="score">0</span> | Level: <span id="lvl-num">1</span>/10</h3>
    </div>

    <div id="game-container">
        <div id="road"></div>
        <div id="car">🚗</div>
        <div id="obstacle-layer">
            <div id="cone" class="obstacle">🚧</div>
        </div>
    </div>

    <div id="ui">
        <div id="sentence-box">Loading...</div>
        <div>
            <button id="mic-btn">🎤 CLICK TO SPEAK</button>
            <button id="restart-btn" style="display:none; background:#3498db;">🔄 TRY AGAIN</button>
        </div>
        <div id="feedback"></div>
    </div>

    <script>
        const levels = [
            { d: "sh", s: "The ship is in the shop", k: ["ship", "shop"] },
            { d: "ch", s: "The chicken has a chin", k: ["chicken", "chin"] },
            { d: "th", s: "The moth is on the path", k: ["moth", "path"] },
            { d: "ph", s: "The dolphin is on the phone", k: ["dolphin", "phone"] },
            { d: "sh", s: "Wash the fish in the dish", k: ["wash", "fish", "dish"] },
            { d: "ch", s: "Check the cheese on the chair", k: ["check", "cheese", "chair"] },
            { d: "th", s: "Think about the thin thorn", k: ["think", "thin", "thorn"] },
            { d: "wh", s: "The white whale is fast", k: ["white", "whale"] },
            { d: "ng", s: "Sing a song for the king", k: ["sing", "song", "king"] },
            { d: "ck", s: "The duck is on the rock", k: ["duck", "rock"] }
        ];

        let current = 0;
        let score = 0;
        const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
        const recognition = new SpeechRecognition();
        recognition.lang = 'en-US';

        const car = document.getElementById('car');
        const road = document.getElementById('road');
        const cone = document.getElementById('cone');
        const sentenceBox = document.getElementById('sentence-box');
        const feedback = document.getElementById('feedback');
        const micBtn = document.getElementById('mic-btn');

        function loadLevel() {
            if (current >= levels.length) {
                sentenceBox.innerHTML = "🏆 YOU FINISHED THE TEST!";
                micBtn.style.display = "none";
                document.getElementById('restart-btn').style.display = "inline-block";
                return;
            }
            
            const data = levels[current];
            let text = data.s.replace(new RegExp(data.d, 'gi'), m => `<span class="highlight">${m}</span>`);
            sentenceBox.innerHTML = text;
            document.getElementById('lvl-num').innerText = current + 1;
            
            // Reset obstacle
            cone.style.opacity = "1";
            cone.style.transform = "scale(1)";
            feedback.innerHTML = "Click the button and read the sentence.";
            micBtn.disabled = false;
        }

        micBtn.onclick = () => {
            recognition.start();
            feedback.innerHTML = "<span style='color:blue'>Listening... Speak now!</span>";
        };

        recognition.onresult = (event) => {
            const result = event.results[0][0].transcript.toLowerCase();
            const data = levels[current];
            const pass = data.k.some(word => result.includes(word)) || result.includes(data.s.toLowerCase());

            if (pass) {
                feedback.innerHTML = `<span class='success'>Correct! I heard "${result}"</span>`;
                win();
            } else {
                feedback.innerHTML = `<span class='error'>I heard "${result}". Try again!</span>`;
            }
        };

        function win() {
            micBtn.disabled = true;
            score += 10;
            document.getElementById('score').innerText = score;

            // ANIMATION: Car stays, Road and Obstacle move left
            cone.style.transform = "scale(0) translateY(-50px)";
            cone.style.opacity = "0";
            
            current++;
            setTimeout(loadLevel, 2000);
        }

        document.getElementById('restart-btn').onclick = () => location.reload();

        loadLevel();
    </script>
</body>
</html>
