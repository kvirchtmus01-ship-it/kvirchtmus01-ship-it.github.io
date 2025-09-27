<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Шаги → Музыка</title>
  <style>
    body { font-family: sans-serif; text-align: center; padding: 20px; background: #f0f8ff; }
    button { padding: 15px 30px; font-size: 18px; margin: 20px; border: none; border-radius: 10px; background: #4CAF50; color: white; cursor: pointer; }
    #status { font-size: 18px; color: #555; }
  </style>
</head>
<body>
  <h1>🚶‍♂️ Шаги → Музыка</h1>
  <button id="start">Разрешить доступ к датчикам</button>
  <div id="status">Жди шагов...</div>

  <script>
    let audioContext;
    let lastStepTime = 0;
    let noteIndex = 0;
    const notes = [261.63, 293.66, 329.63, 349.23, 392.00, 440.00, 493.88];
    const STEP_THRESHOLD = 12;
    const MIN_STEP_INTERVAL = 300;

    function playNote(frequency) {
      if (!audioContext) audioContext = new (window.AudioContext || window.webkitAudioContext)();
      const o = audioContext.createOscillator(), g = audioContext.createGain();
      o.type = 'sine'; o.frequency.value = frequency;
      g.gain.value = 0.2; g.gain.exponentialRampToValueAtTime(0.001, audioContext.currentTime + 0.3);
      o.connect(g); g.connect(audioContext.destination);
      o.start(); o.stop(audioContext.currentTime + 0.3);
    }

    function onMotion(e) {
      const now = Date.now();
      if (now - lastStepTime < MIN_STEP_INTERVAL) return;
      const a = e.accelerationIncludingGravity;
      if (!a) return;
      const f = Math.abs(a.x) + Math.abs(a.y) + Math.abs(a.z);
      if (f > STEP_THRESHOLD) {
        playNote(notes[noteIndex % notes.length]);
        noteIndex++;
        document.getElementById('status').innerText = '🎵 Шаг! Нота: ' + Math.round(notes[noteIndex-1]) + ' Гц';
        lastStepTime = now;
      }
    }

    document.getElementById('start').onclick = async () => {
      try {
        if (typeof DeviceMotionEvent.requestPermission === 'function') {
          const p = await DeviceMotionEvent.requestPermission();
          if (p !== 'granted') throw new Error('Доступ запрещён');
        }
        window.addEventListener('devicemotion', onMotion);
        document.getElementById('status').innerText = '✅ Готово! Иди и делай шаги!';
      } catch (err) {
        document.getElementById('status').innerText = '❌ ' + err.message;
      }
    };
  </script>
</body>
</html>
