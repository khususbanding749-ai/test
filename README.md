<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>GrenXharimau AI Voice</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    font-family: sans-serif;
    background: #0d1117;
    color: #e6edf3;
    height: 100vh;
    display: flex;
    flex-direction: column;
  }
  header {
    background: #1f6feb;
    padding: 12px;
    text-align: center;
    font-weight: bold;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 12px 16px;
  }
  #testVoice {
    background: #3fb950;
    border: none;
    color: #fff;
    padding: 6px 12px;
    border-radius: 20px;
    font-size: 12px;
    cursor: pointer;
  }
  #chat {
    flex: 1;
    overflow-y: auto;
    padding: 12px;
    display: flex;
    flex-direction: column;
    gap: 10px;
  }
  .msg {
    max-width: 85%;
    padding: 10px 14px;
    border-radius: 16px;
    font-size: 14px;
    line-height: 1.5;
    white-space: pre-wrap;
    word-wrap: break-word;
  }
  .user {
    align-self: flex-end;
    background: #1f6feb;
    color: #fff;
    border-bottom-right-radius: 4px;
  }
  .ai {
    align-self: flex-start;
    background: #161b22;
    border: 1px solid #30363d;
    border-bottom-left-radius: 4px;
  }
  .error {
    align-self: flex-start;
    background: #2d1418;
    border: 1px solid #f85149;
    color: #ff7b72;
    border-bottom-left-radius: 4px;
  }
  footer {
    display: flex;
    gap: 8px;
    padding: 10px;
    background: #161b22;
    border-top: 1px solid #30363d;
  }
  #input {
    flex: 1;
    padding: 12px;
    border-radius: 20px;
    border: 1px solid #30363d;
    background: #0d1117;
    color: #e6edf3;
    font-size: 14px;
    outline: none;
    min-width: 0;
  }
  #send {
    padding: 0 20px;
    border: none;
    border-radius: 20px;
    background: #1f6feb;
    color: #fff;
    font-size: 15px;
    cursor: pointer;
    font-weight: 600;
  }
  #send:disabled { opacity: 0.5; }
</style>
</head>
<body>

<header>
  <span>🐯 GrenXharimau AI</span>
  <button id="testVoice">🔊 Test Suara</button>
</header>

<div id="chat">
  <div class="msg ai" id="boot">
    Halo! Aku GrenXharimau AI 🐯<br><br>
    <b>Kalau suara gak keluar:</b><br>
    1. Tap di mana aja dulu di halaman<br>
    2. Tap tombol 🔊 Test Suara di atas<br>
    3. Cek volume HP
  </div>
</div>

<footer>
  <input id="input" type="text" placeholder="Tulis pesan..." autocomplete="off" />
  <button id="send">Kirim</button>
</footer>

<script>
  const API = "http://ryzzgantenk.ryuuuprvt.my.id:4318/salam";
  const chat = document.getElementById("chat");
  const input = document.getElementById("input");
  const sendBtn = document.getElementById("send");
  const testBtn = document.getElementById("testVoice");
  const boot = document.getElementById("boot");

  // ===== CEK SUPPORT =====
  const hasTTS = "speechSynthesis" in window;
  console.log("Text-to-Speech supported:", hasTTS);

  // ===== TEST SUARA =====
  function testSuara() {
    if (!hasTTS) {
      alert("Browser kamu gak support Text-to-Speech. Coba pakai Chrome!");
      return;
    }
    speechSynthesis.cancel();
    const u = new SpeechSynthesisUtterance("Tes suara berhasil. Halo, aku GrenXharimau AI.");
    u.lang = "id-ID";
    u.rate = 1.0;
    u.pitch = 1.1;

    // Cari voice Indonesia
    const voices = speechSynthesis.getVoices();
    console.log("Total voices:", voices.length);
    const idVoice = voices.find(v => v.lang.startsWith("id"));
    if (idVoice) {
      u.voice = idVoice;
      console.log("Pakai voice:", idVoice.name);
    } else {
      console.warn("Voice Indonesia gak ada, pakai default");
    }

    u.onerror = (e) => alert("Error suara: " + e.error);
    speechSynthesis.speak(u);
  }

  testBtn.onclick = testSuara;

  // ===== SPEAK =====
  function speak(text) {
    if (!hasTTS) return;
    speechSynthesis.cancel();

    const clean = text
      .replace(/[\u{1F300}-\u{1FAFF}]/gu, "")
      .replace(/[*_~`#]/g, "")
      .replace(/\n+/g, ". ")
      .trim();

    const u = new SpeechSynthesisUtterance(clean);
    u.lang = "id-ID";
    u.rate = 1.0;
    u.pitch = 1.1;

    const voices = speechSynthesis.getVoices();
    const idVoice = voices.find(v => v.lang.startsWith("id"));
    if (idVoice) u.voice = idVoice;

    speechSynthesis.speak(u);
  }

  // ===== ADD MSG =====
  function addMsg(text, type) {
    const div = document.createElement("div");
    div.className = "msg " + type;
    div.textContent = text;
    chat.appendChild(div);
    chat.scrollTop = chat.scrollHeight;
  }

  // ===== SEND =====
  async function send() {
    const pesan = input.value.trim();
    if (!pesan) return;

    if (boot && boot.parentNode) boot.remove();

    addMsg(pesan, "user");
    input.value = "";
    sendBtn.disabled = true;

    try {
      const res = await fetch(API + "?pesan=" + encodeURIComponent(pesan));
      const data = await res.json();

      if (data.status) {
        addMsg(data.result, "ai");
        speak(data.result);
      } else {
        addMsg("⚠️ " + data.result, "error");
      }
    } catch (err) {
      addMsg("⚠️ Gagal konek: " + err.message, "error");
    } finally {
      sendBtn.disabled = false;
      input.focus();
    }
  }

  sendBtn.onclick = send;
  input.onkeydown = (e) => { if (e.key === "Enter") send(); };

  // Preload voices
  if (hasTTS) {
    speechSynthesis.getVoices();
    speechSynthesis.onvoiceschanged = () => {
      console.log("Voices loaded:", speechSynthesis.getVoices().length);
    };
  }
</script>
</body>
</html>
