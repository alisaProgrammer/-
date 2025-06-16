<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <title>Juego de memoria con emojis</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>
    body {
      background: #222;
      color: #eee;
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 20px;
      text-align: center;
    }

    h1 {
      margin-bottom: 10px;
    }

    #level-select {
      margin-bottom: 15px;
    }

    #game {
      display: grid;
      gap: 8px;
      justify-content: center;
      margin: 0 auto;
    }

    .card {
      background: #444;
      border-radius: 10px;
      cursor: pointer;
      display: flex;
      justify-content: center;
      align-items: center;
      font-size: 32px;
      user-select: none;
      transition: transform 0.3s;
      aspect-ratio: 1 / 1;
    }

    .card.flipped {
      background: #61dafb;
      cursor: default;
      transform: rotateY(180deg);
    }

    .card.matched {
      background: #7fff00;
      cursor: default;
    }

    #status {
      margin-top: 15px;
      font-size: 18px;
    }

    #timer {
      font-size: 18px;
      margin-bottom: 10px;
    }

    #congrats-img {
      display: none;
      position: fixed;
      top: 0;
      left: 0;
      width: 100vw;
      height: 100vh;
      object-fit: cover;
      z-index: 1000;
      background: #000;
      animation: pop 0.2s ease-out;
      filter: contrast(1.3) brightness(1.2);
      image-rendering: pixelated;
      border-radius: 0;
      box-shadow: 0 0 20px #fff;
    }

    #back-to-game {
      display: none;
      z-index: 1001;
      position: fixed;
      top: 20px;
      right: 20px;
      padding: 10px 20px;
      font-size: 16px;
      background: #444;
      color: #fff;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      box-shadow: 0 0 10px #000;
    }

    @keyframes pop {
      0% { transform: scale(0); opacity: 0; }
      100% { transform: scale(1); opacity: 1; }
    }

    @media (max-width: 600px) {
      .card {
        font-size: 24px;
      }
    }
  </style>
</head>
<body>

  <h1>Juego de memoria — ¡encuentra las parejas!</h1>
  <div id="level-select">
    <label for="level">Dificultad: </label>
    <select id="level">
      <option value="5">Muy fácil (5 parejas)</option>
      <option value="10">Fácil (10 parejas)</option>
      <option value="15">Media (15 parejas)</option>
      <option value="20">Difícil (20 parejas)</option>
      <option value="25">Muy difícil (25 parejas)</option>
    </select>
    <button onclick="startGame()">Comenzar juego</button>
  </div>

  <div id="timer">⏱ Tiempo: <span id="time">0</span> seg</div>
  <div id="game"></div>
  <p id="status"></p>

  <img id="congrats-img" src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcStOPi49P1wRC5TOSZGj8zSmRYyV-ebX9NMFz-MVpQKz2ThiHOKiS47V-gKvwtu522IYvU&usqp=CAU" alt="¡Sorpresa!">
  <button id="back-to-game">🔁 Volver al juego</button>

  <script>
    const symbols = ['😀','😁','😂','🤣','😃','😄','😅','😆','😉','😊','😋','😎','😍','😘','😗','😙','😚','🙂','🤗','🤩','🤔','🤨','😐','😑','😶','🙄','😏','😣','😥','😮','😱','😳','😴','😡','😇','🥳','🥶','😈','👻','🤖','🎃','🐵','🐶','🐱','🦊','🐰','🦄','🐯','🐸','🐷'];

    let cards = [], flippedCards = [], matchedCount = 0, timer, seconds = 0;
    const game = document.getElementById('game');
    const status = document.getElementById('status');
    const timeDisplay = document.getElementById('time');
    const congratsImg = document.getElementById('congrats-img');
    const backToGameBtn = document.getElementById('back-to-game');

    function shuffle(array) {
      for (let i = array.length -1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
    }

    function startGame() {
      clearInterval(timer);
      seconds = 0;
      timeDisplay.textContent = '0';
      status.textContent = '';
      congratsImg.style.display = 'none';
      backToGameBtn.style.display = 'none';
      game.innerHTML = '';
      flippedCards = [];
      matchedCount = 0;

      const level = parseInt(document.getElementById('level').value);
      const selectedSymbols = symbols.slice(0, level);
      cards = [...selectedSymbols, ...selectedSymbols];
      shuffle(cards);

      const columns = Math.ceil(Math.sqrt(level * 2));
      game.style.gridTemplateColumns = `repeat(${columns}, minmax(50px, 1fr))`;

      for (let symbol of cards) {
        const card = document.createElement('div');
        card.classList.add('card');
        card.dataset.symbol = symbol;
        card.textContent = '';
        card.addEventListener('click', onCardClick);
        game.appendChild(card);
      }

      timer = setInterval(() => {
        seconds++;
        timeDisplay.textContent = seconds;
      }, 1000);
    }

    function onCardClick(e) {
      const card = e.target;
      if (flippedCards.length === 2 || card.classList.contains('flipped') || card.classList.contains('matched')) return;

      card.classList.add('flipped');
      card.textContent = card.dataset.symbol;
      flippedCards.push(card);

      if (flippedCards.length === 2) {
        if (flippedCards[0].dataset.symbol === flippedCards[1].dataset.symbol) {
          flippedCards.forEach(c => c.classList.add('matched'));
          matchedCount += 2;
          flippedCards = [];

          if (matchedCount === cards.length) {
            clearInterval(timer);
            setTimeout(() => {
              congratsImg.style.display = 'block';
              backToGameBtn.style.display = 'block';
              status.textContent = `🎉 ¡Bien hecho! ¡Encontraste todas las parejas en ${seconds} segundos!`;
            }, 300);
          }
        } else {
          setTimeout(() => {
            flippedCards.forEach(c => {
              c.classList.remove('flipped');
              c.textContent = '';
            });
            flippedCards = [];
          }, 1000);
        }
      }
    }

    backToGameBtn.addEventListener('click', () => {
      congratsImg.style.display = 'none';
      backToGameBtn.style.display = 'none';
      startGame();
    });
  </script>

</body>
</html>
# -
