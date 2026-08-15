# German-app
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
  <title>German Flashcards</title>
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
    body { background: #121212; color: #fff; height: 100vh; display: flex; flex-direction: column; justify-content: space-between; overflow: hidden; }
    
    header { padding: 15px; text-align: center; background: #1e1e1e; border-bottom: 1px solid #333; font-weight: bold; font-size: 1.2rem; }
    .content { flex: 1; overflow-y: auto; padding: 20px; display: flex; flex-direction: column; }
    .tab-content { display: none; height: 100%; flex-direction: column; }
    .tab-content.active { display: flex; }

    nav { display: flex; background: #1e1e1e; border-top: 1px solid #333; }
    nav button { flex: 1; padding: 15px 5px; background: none; border: none; color: #888; font-size: 0.85rem; font-weight: 600; }
    nav button.active { color: #4caf50; border-top: 2px solid #4caf50; }

    input, select, button { width: 100%; padding: 12px; margin-bottom: 10px; border-radius: 8px; border: 1px solid #333; background: #222; color: #fff; font-size: 1rem; }
    button.primary { background: #4caf50; color: #000; font-weight: bold; border: none; }
    button.secondary { background: #333; border: none; }
    .mic-btn { background: #2196f3; color: #fff; border: none; }

    /* Flashcard Studio & Playlists */
    .card-list { display: flex; flex-direction: column; gap: 10px; margin-top: 15px; }
    .card-item { background: #1e1e1e; padding: 12px; border-radius: 8px; border: 1px solid #333; display: flex; justify-content: space-between; align-items: center; }

    /* Study Mode */
    .flip-card { perspective: 1000px; width: 100%; height: 220px; margin-bottom: 20px; }
    .flip-card-inner { position: relative; width: 100%; height: 100%; text-align: center; transition: transform 0.6s; transform-style: preserve-3d; border-radius: 12px; }
    .flip-card.flipped .flip-card-inner { transform: rotateY(180deg); }
    .flip-card-front, .flip-card-back { position: absolute; width: 100%; height: 100%; backface-visibility: hidden; border-radius: 12px; background: #1e1e1e; border: 1px solid #333; display: flex; flex-direction: column; justify-content: center; align-items: center; padding: 15px; }
    .flip-card-back { transform: rotateY(180deg); background: #252525; }
    .sentence { font-size: 0.9rem; color: #aaa; margin-top: 10px; font-style: italic; }

    .controls { display: flex; gap: 10px; margin-top: auto; }
    .btn-red { background: #f44336; color: #fff; border: none; }
    .btn-green { background: #4caf50; color: #fff; border: none; }

    /* Shuffle Animation */
    .shuffle-anim { animation: shuffle 0.4s ease-in-out; }
    @keyframes shuffle {
      0% { transform: translateX(0) scale(1); }
      50% { transform: translateX(100px) scale(0.8) rotate(10deg); }
      100% { transform: translateX(0) scale(1); }
    }

    /* Stats */
    .stat-box { background: #1e1e1e; border: 1px solid #333; border-radius: 12px; padding: 20px; text-align: center; margin-bottom: 15px; }
    .stat-number { font-size: 3rem; font-weight: bold; color: #4caf50; }
  </style>
</head>
<body>

  <header id="header-title">Cards & Playlists</header>

  <div class="content">
    <!-- TAB 1: CARDS & PLAYLISTS -->
    <div id="tab-cards" class="tab-content active">
      <input type="text" id="playlist-name" placeholder="New Playlist Name">
      <button class="primary" onclick="createPlaylist()">Create Playlist</button>
      
      <hr style="border-color:#333; margin: 15px 0;">

      <select id="select-playlist-card"></select>
      <input type="text" id="word-de" placeholder="German Word">
      <div style="display:flex; gap: 5px;">
        <input type="text" id="word-en" placeholder="English Translation">
        <button class="mic-btn" style="width: 30%;" onclick="startDictation('word-en')">🎤</button>
      </div>
      <input type="text" id="word-sentence" placeholder="Example Sentence (German)">
      <button class="primary" onclick="addCard()">Add Card</button>

      <div class="card-list" id="card-list-display"></div>
    </div>

    <!-- TAB 2: STUDY / PLAYLIST MODE -->
    <div id="tab-study" class="tab-content">
      <select id="select-study-playlist" onchange="loadStudyPlaylist()"></select>
      <button class="secondary" onclick="shuffleStudyCards()">🔀 Shuffle Cards</button>
      
      <div id="study-area" style="flex:1; display:flex; flex-direction:column; justify-content:center;">
        <div class="flip-card" id="study-card" onclick="this.classList.toggle('flipped')">
          <div class="flip-card-inner">
            <div class="flip-card-front">
              <h2 id="study-de">Select a Playlist</h2>
              <p class="sentence" id="study-sentence"></p>
            </div>
            <div class="flip-card-back">
              <h2 id="study-en">...</h2>
            </div>
          </div>
        </div>

        <div class="controls">
          <button class="btn-red" onclick="handleCardAnswer(false)">I Don't Remember</button>
          <button class="btn-green" onclick="handleCardAnswer(true)">I Remember</button>
        </div>
      </div>
    </div>

    <!-- TAB 3: TEST MODE -->
    <div id="tab-test" class="tab-content">
      <select id="select-test-playlist" onchange="startTest()"></select>
      <div id="test-area" style="flex:1; display:flex; flex-direction:column; justify-content:center;">
        <h2 id="test-prompt" style="text-align:center; margin-bottom: 10px;">Select Playlist</h2>
        <p id="test-sentence" class="sentence" style="text-align:center; margin-bottom: 20px;"></p>
        <input type="text" id="test-input" placeholder="Type English Translation">
        <button class="primary" onclick="submitTestAnswer()">Submit Answer</button>
        <p id="test-feedback" style="text-align:center; margin-top: 15px; font-weight: bold;"></p>
      </div>
    </div>

    <!-- TAB 4: STATISTICS -->
    <div id="tab-stats" class="tab-content">
      <div class="stat-box">
        <div class="stat-number" id="stat-learned-count">0</div>
        <div>Words Learned</div>
      </div>
      <div class="stat-box">
        <div class="stat-number" id="stat-total-count" style="color: #2196f3;">0</div>
        <div>Total Cards Created</div>
      </div>
    </div>
  </div>

  <nav>
    <button class="active" onclick="switchTab('cards', this)">Cards</button>
    <button onclick="switchTab('study', this)">Study</button>
    <button onclick="switchTab('test', this)">Test</button>
    <button onclick="switchTab('stats', this)">Stats</button>
  </nav>

  <script>
    let appData = JSON.parse(localStorage.getItem('de_app_data')) || {
      playlists: ['Default'],
      cards: [],
      learnedWords: []
    };

    let activeStudyQueue = [];
    let currentTestCard = null;

    function saveData() {
      localStorage.setItem('de_app_data', JSON.stringify(appData));
      updateUI();
    }

    function switchTab(tabId, btn) {
      document.querySelectorAll('.tab-content').forEach(t => t.classList.remove('active'));
      document.querySelectorAll('nav button').forEach(b => b.classList.remove('active'));
      document.getElementById('tab-' + tabId).classList.add('active');
      btn.classList.add('active');

      if (tabId === 'study') loadStudyPlaylist();
      if (tabId === 'test') startTest();
      if (tabId === 'stats') renderStats();
    }

    function createPlaylist() {
      const name = document.getElementById('playlist-name').value.trim();
      if (name && !appData.playlists.includes(name)) {
        appData.playlists.push(name);
        document.getElementById('playlist-name').value = '';
        saveData();
      }
    }

    function addCard() {
      const de = document.getElementById('word-de').value.trim();
      const en = document.getElementById('word-en').value.trim();
      const sentence = document.getElementById('word-sentence').value.trim();
      const playlist = document.getElementById('select-playlist-card').value;

      if (de && en && playlist) {
        appData.cards.push({ id: Date.now(), de, en, sentence, playlist });
        document.getElementById('word-de').value = '';
        document.getElementById('word-en').value = '';
        document.getElementById('word-sentence').value = '';
        saveData();
      }
    }

    function startDictation(targetId) {
      if ('webkitSpeechRecognition' in window || 'SpeechRecognition' in window) {
        const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
        const recognition = new SpeechRecognition();
        recognition.lang = 'en-US';
        recognition.start();
        recognition.onresult = (event) => {
          document.getElementById(targetId).value = event.results[0][0].transcript;
        };
      } else {
        alert('Speech recognition is not supported on this browser.');
      }
    }

    function updateUI() {
      const selects = ['select-playlist-card', 'select-study-playlist', 'select-test-playlist'];
      selects.forEach(id => {
        const sel = document.getElementById(id);
        const val = sel.value;
        sel.innerHTML = appData.playlists.map(p => `<option value="${p}">${p}</option>`).join('');
        if (val && appData.playlists.includes(val)) sel.value = val;
      });

      const cardList = document.getElementById('card-list-display');
      cardList.innerHTML = appData.cards.map(c => `
        <div class="card-item">
          <div>
            <strong>${c.de}</strong> - ${c.en}<br>
            <small style="color:#888;">${c.playlist} ${c.sentence ? '| ' + c.sentence : ''}</small>
          </div>
          <button style="width:auto; padding:5px 10px; margin:0;" onclick="deleteCard(${c.id})">✕</button>
        </div>
      `).join('');
    }

    function deleteCard(id) {
      appData.cards = appData.cards.filter(c => c.id !== id);
      appData.learnedWords = appData.learnedWords.filter(wId => wId !== id);
      saveData();
    }

    /* Study Mode */
    function loadStudyPlaylist() {
      const selected = document.getElementById('select-study-playlist').value;
      activeStudyQueue = appData.cards.filter(c => c.playlist === selected);
      renderStudyCard();
    }

    function shuffleStudyCards() {
      const cardEl = document.getElementById('study-card');
      cardEl.classList.add('shuffle-anim');
      setTimeout(() => cardEl.classList.remove('shuffle-anim'), 400);

      for (let i = activeStudyQueue.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [activeStudyQueue[i], activeStudyQueue[j]] = [activeStudyQueue[j], activeStudyQueue[i]];
      }
      renderStudyCard();
    }

    function renderStudyCard() {
      const card = document.getElementById('study-card');
      card.classList.remove('flipped');
      
      if (activeStudyQueue.length === 0) {
        document.getElementById('study-de').innerText = "Playlist Complete!";
        document.getElementById('study-sentence').innerText = "";
        document.getElementById('study-en').innerText = "Nice job!";
        return;
      }

      const current = activeStudyQueue[0];
      document.getElementById('study-de').innerText = current.de;
      document.getElementById('study-sentence').innerText = current.sentence || "";
      document.getElementById('study-en').innerText = current.en;
    }

    function handleCardAnswer(remembered) {
      if (activeStudyQueue.length === 0) return;

      const current = activeStudyQueue.shift();
      if (remembered) {
        if (!appData.learnedWords.includes(current.id)) {
          appData.learnedWords.push(current.id);
          saveData();
        }
      } else {
        activeStudyQueue.push(current);
      }
      renderStudyCard();
    }

    /* Test Mode */
    function startTest() {
      const selected = document.getElementById('select-test-playlist').value;
      const cards = appData.cards.filter(c => c.playlist === selected);
      document.getElementById('test-feedback').innerText = "";
      document.getElementById('test-input').value = "";

      if (cards.length === 0) {
        document.getElementById('test-prompt').innerText = "No cards in playlist";
        document.getElementById('test-sentence').innerText = "";
        currentTestCard = null;
        return;
      }

      currentTestCard = cards[Math.floor(Math.random() * cards.length)];
      document.getElementById('test-prompt').innerText = currentTestCard.de;
      document.getElementById('test-sentence').innerText = currentTestCard.sentence || "";
    }

    function submitTestAnswer() {
      if (!currentTestCard) return;
      const input = document.getElementById('test-input').value.trim().toLowerCase();
      const expected = currentTestCard.en.trim().toLowerCase();
      const feedback = document.getElementById('test-feedback');

      if (input === expected) {
        feedback.style.color = "#4caf50";
        feedback.innerText = "Correct!";
        if (!appData.learnedWords.includes(currentTestCard.id)) {
          appData.learnedWords.push(currentTestCard.id);
          saveData();
        }
        setTimeout(startTest, 1200);
      } else {
        feedback.style.color = "#f44336";
        feedback.innerText = `Wrong! Answer: ${currentTestCard.en}`;
      }
    }

    function renderStats() {
      document.getElementById('stat-learned-count').innerText = appData.learnedWords.length;
      document.getElementById('stat-total-count').innerText = appData.cards.length;
    }

    updateUI();
  </script>
</body>
</html>
