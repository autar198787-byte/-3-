<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>English Words Match-3 Game</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        body {
            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            color: white;
            padding: 15px;
        }

        .game-card {
            background: rgba(255, 255, 255, 0.95);
            border-radius: 20px;
            padding: 25px;
            box-shadow: 0 15px 35px rgba(0,0,0,0.3);
            width: 100%;
            max-width: 500px;
            text-align: center;
            color: #333;
        }

        h1 {
            color: #2c3e50;
            font-size: 26px;
            margin-bottom: 5px;
        }

        p.subtitle {
            color: #7f8c8d;
            font-size: 14px;
            margin-bottom: 15px;
        }

        /* Category Badge */
        .category-badge {
            display: inline-block;
            background: #e74c3c;
            color: white;
            padding: 4px 12px;
            border-radius: 15px;
            font-size: 12px;
            font-weight: bold;
            margin-bottom: 12px;
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }

        /* Dashboards */
        .stats-bar {
            display: flex;
            justify-content: space-around;
            background: #f1f2f6;
            padding: 12px;
            border-radius: 12px;
            margin-bottom: 20px;
            font-weight: bold;
        }

        .stat-item {
            display: flex;
            flex-direction: column;
        }

        .stat-value {
            font-size: 20px;
            color: #2980b9;
        }

        /* Grid Setup */
        .grid {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 10px;
            margin-bottom: 20px;
            perspective: 1000px;
        }

        /* Flip Card Effect */
        .card {
            aspect-ratio: 1;
            background: transparent;
            cursor: pointer;
            border-radius: 10px;
            perspective: 1000px;
        }

        .card-inner {
            position: relative;
            width: 100%;
            height: 100%;
            text-align: center;
            transition: transform 0.4s;
            transform-style: preserve-3d;
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0,0,0,0.15);
        }

        .card.flipped .card-inner {
            transform: rotateY(180deg);
        }

        .card-front, .card-back {
            position: absolute;
            width: 100%;
            height: 100%;
            -webkit-backface-visibility: hidden;
            backface-visibility: hidden;
            display: flex;
            align-items: center;
            justify-content: center;
            border-radius: 10px;
            font-weight: bold;
            font-size: 15px;
        }

        /* ด้านหลังการ์ด (ซ่อนคำศัพท์) */
        .card-front {
            background: linear-gradient(135deg, #ff9a9e 0%, #fecfef 99%);
            color: #fff;
            font-size: 24px;
            border: 2px solid #fff;
        }

        /* ด้านหน้าการ์ด (เปิดเห็นคำศัพท์) */
        .card-back {
            background: #3498db;
            color: white;
            transform: rotateY(180deg);
            word-break: break-word;
            padding: 5px;
        }

        .card.selected .card-back {
            background: #e67e22;
        }

        .card.matched {
            visibility: hidden;
            opacity: 0;
            transition: opacity 0.5s, visibility 0.5s;
            pointer-events: none;
        }

        /* Controls */
        .btn {
            background: #2ecc71;
            color: white;
            border: none;
            padding: 12px 30px;
            font-size: 16px;
            border-radius: 25px;
            cursor: pointer;
            font-weight: bold;
            box-shadow: 0 4px 15px rgba(46, 204, 113, 0.4);
            transition: all 0.2s;
        }

        .btn:hover {
            background: #27ae60;
            transform: translateY(-2px);
        }

        .btn:active {
            transform: translateY(0);
        }
    </style>
</head>
<body>

    <div class="game-card">
        <h1>🎮 Match-3 Word Game</h1>
        <p class="subtitle">จับคู่คำศัพท์ภาษาอังกฤษที่เหมือนกัน 3 ใบ!</p>
        <div id="category" class="category-badge">หมวดหมู่: Animals</div>

        <div class="stats-bar">
            <div class="stat-item">
                <span>ด่าน (Level)</span>
                <span id="level" class="stat-value">1/10</span>
            </div>
            <div class="stat-item">
                <span>คะแนน (Score)</span>
                <span id="score" class="stat-value">0</span>
            </div>
            <div class="stat-item">
                <span>เวลา (Time)</span>
                <span id="timer" class="stat-value">45s</span>
            </div>
        </div>

        <div class="grid" id="gameGrid"></div>

        <button class="btn" onclick="restartGame()">เริ่มเกมใหม่</button>
    </div>

    <script>
        // คลังคำศัพท์ 10 ด่าน พร้อมหมวดหมู่
        const levelsData = [
            { level: 1, category: "Animals (สัตว์)", words: ['Cat', 'Dog', 'Lion', 'Bear'] },
            { level: 2, category: "Fruits (ผลไม้)", words: ['Apple', 'Banana', 'Mango', 'Grape'] },
            { level: 3, category: "Colors (สี)", words: ['Red', 'Blue', 'Green', 'Yellow'] },
            { level: 4, category: "School (โรงเรียน)", words: ['Book', 'Pen', 'Desk', 'Class'] },
            { level: 5, category: "Food (อาหาร)", words: ['Pizza', 'Bread', 'Rice', 'Soup'] },
            { level: 6, category: "Body (ร่างกาย)", words: ['Hand', 'Head', 'Eye', 'Nose'] },
            { level: 7, category: "Nature (ธรรมชาติ)", words: ['Tree', 'River', 'Star', 'Moon'] },
            { level: 8, category: "Vehicle (ยานพาหนะ)", words: ['Car', 'Train', 'Ship', 'Plane'] },
            { level: 9, category: "Weather (สภาพอากาศ)", words: ['Rain', 'Wind', 'Cloud', 'Snow'] },
            { level: 10, category: "Feelings (ความรู้สึก)", words: ['Happy', 'Smile', 'Brave', 'Smart'] }
        ];

        let currentLevel = 0; // 0 คือ Level 1
        let score = 0;
        let timeLeft = 45;
        let timerInterval = null;
        let selectedCards = [];
        let isProcessing = false;

        // Web Audio API Synthesizer (ระดับเสียงดังชัดเจน)
        let audioCtx = null;

        function getAudioContext() {
            if (!audioCtx) {
                audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            }
            if (audioCtx.state === 'suspended') {
                audioCtx.resume();
            }
            return audioCtx;
        }

        function playNote(freq, type = 'sine', duration = 0.1, startTime = 0, gainVal = 0.5) {
            const ctx = getAudioContext();
            const osc = ctx.createOscillator();
            const gain = ctx.createGain();
            
            osc.type = type;
            osc.frequency.setValueAtTime(freq, ctx.currentTime + startTime);
            
            gain.gain.setValueAtTime(gainVal, ctx.currentTime + startTime);
            gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + startTime + duration);
            
            osc.connect(gain);
            gain.connect(ctx.destination);
            
            osc.start(ctx.currentTime + startTime);
            osc.stop(ctx.currentTime + startTime + duration);
        }

        function playFlipSound() {
            playNote(520, 'square', 0.1, 0, 0.4);
        }

        function playMatchSound() {
            playNote(523.25, 'triangle', 2000, 2000, 2000); // C5
            playNote(659.25, 'triangle', 2000,2000,2000); // E5
            playNote(783.99, 'triangle', 2000, 2000, 2000); // G5
        }

        function playWrongSound() {
            playNote(220, 'sawtooth', 0.18, 0.0, 0.5);
            playNote(175, 'sawtooth', 0.3, 0.12, 0.5);
        }

        function playWinSound() {
            playNote(523.25, 'triangle', 0.15, 0.0, 0.6); // C5
            playNote(659.25, 'triangle', 0.15, 0.1, 0.6); // E5
            playNote(783.99, 'triangle', 0.15, 0.2, 0.6); // G5
            playNote(1046.50, 'triangle', 0.5, 0.3, 0.7); // C6
        }

        function playGameOverSound() {
            playNote(300, 'sawtooth', 0.2, 0.0, 0.5);
            playNote(250, 'sawtooth', 0.2, 0.2, 0.5);
            playNote(200, 'sawtooth', 0.2, 0.4, 0.5);
            playNote(150, 'sawtooth', 0.6, 0.6, 0.5);
        }

        function startTimer() {
            clearInterval(timerInterval);
            timerInterval = setInterval(() => {
                timeLeft--;
                document.getElementById('timer').innerText = timeLeft + 's';
                
                if (timeLeft <= 0) {
                    clearInterval(timerInterval);
                    playGameOverSound();
                    setTimeout(() => {
                        alert('⏰ หมดเวลาแล้ว! คะแนนรวมของคุณคือ: ' + score);
                        restartGame();
                    }, 700);
                }
            }, 1000);
        }

        function initLevel() {
            const grid = document.getElementById('gameGrid');
            grid.innerHTML = '';
            selectedCards = [];
            isProcessing = false;

            const currentData = levelsData[currentLevel];
            document.getElementById('level').innerText = `${currentData.level}/${levelsData.length}`;
            document.getElementById('category').innerText = `หมวดหมู่: ${currentData.category}`;
            
            let boardWords = [...currentData.words, ...currentData.words, ...currentData.words];
            boardWords.sort(() => Math.random() - 0.5);

            boardWords.forEach((word) => {
                const card = document.createElement('div');
                card.classList.add('card');
                card.dataset.word = word;

                card.innerHTML = `
                    <div class="card-inner">
                        <div class="card-front">❓</div>
                        <div class="card-back">${word}</div>
                    </div>
                `;

                card.onclick = () => flipCard(card);
                grid.appendChild(card);
            });
        }

        function flipCard(card) {
            if (isProcessing || card.classList.contains('flipped') || card.classList.contains('matched')) return;

            playFlipSound();
            card.classList.add('flipped', 'selected');
            selectedCards.push(card);

            if (selectedCards.length === 3) {
                isProcessing = true;
                const [c1, c2, c3] = selectedCards;

                if (c1.dataset.word === c2.dataset.word && c2.dataset.word === c3.dataset.word) {
                    setTimeout(() => {
                        playMatchSound();
                        selectedCards.forEach(c => c.classList.add('matched'));
                        score += 30;
                        document.getElementById('score').innerText = score;
                        selectedCards = [];
                        isProcessing = false;
                        checkWin();
                    }, 400);
                } else {
                    setTimeout(() => {
                        playWrongSound();
                        selectedCards.forEach(c => c.classList.remove('flipped', 'selected'));
                        selectedCards = [];
                        isProcessing = false;
                    }, 800);
                }
            }
        }

        function checkWin() {
            const remainingCards = document.querySelectorAll('.card:not(.matched)');
            if (remainingCards.length === 0) {
                setTimeout(() => {
                    playWinSound();
                    setTimeout(() => {
                        if (currentLevel + 1 < levelsData.length) {
                            alert(`🎉 ผ่านด่านที่ ${currentLevel + 1} แล้ว! ปลดล็อกด่านถัดไป`);
                            currentLevel++;
                            timeLeft += 20; // เพิ่มเวลา 20 วินาทีเมื่อผ่านด่าน
                            initLevel();
                        } else {
                            alert(`🏆 ยินดีด้วย! คุณชนะครบทั้ง ${levelsData.length} ด่านแล้ว! คะแนนรวม: ${score}`);
                            restartGame();
                        }
                    }, 500);
                }, 300);
            }
        }

        function restartGame() {
            currentLevel = 0;
            score = 0;
            timeLeft = 45;
            document.getElementById('score').innerText = score;
            document.getElementById('timer').innerText = timeLeft + 's';
            initLevel();
            startTimer();
        }

        window.onload = () => {
            restartGame();
        };
    </script>
</body>
</html>

