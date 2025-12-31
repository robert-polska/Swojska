<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>IXH – Presostaty</title>

<style>
/* ====== GLOBAL ====== */
body {
  margin: 0;
  padding: 0;
  font-family: Arial, Helvetica, sans-serif;
  background: #f2f2f2;
}

/* ====== PANEL GŁÓWNY ====== */
.panel {
  max-width: 1100px;
  margin: 20px auto;
  background: #ffffff;
  border: 3px solid #2c2c2c;
  border-radius: 6px;
  box-shadow: 0 0 12px rgba(0,0,0,0.2);
}

/* ====== NAGŁÓWEK ====== */
.header {
  background: #2c2c2c;
  color: #ffffff;
  padding: 12px 16px;
  font-size: 20px;
  font-weight: bold;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.header .title {
  letter-spacing: 1px;
}

.header .alarms {
  font-size: 18px;
}

/* ====== SIATKA KAFELKÓW ====== */
.grid {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 10px;
  padding: 16px;
}

/* ====== KAFEL ====== */
.tile {
  border: 2px solid #2c2c2c;
  border-radius: 4px;
  height: 70px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 18px;
  font-weight: bold;
  background: #cccccc;
  color: #000000;
}

/* ====== STANY ====== */
.tile.ok {
  background: #2ecc71;   /* zielony */
}

.tile.alarm {
  background: #e74c3c;   /* czerwony */
  color: #ffffff;
}

/* ====== HISTORIA ====== */
.history {
  border-top: 3px solid #2c2c2c;
  padding: 10px;
}

.history-title {
  font-weight: bold;
  margin-bottom: 6px;
}

.history-box {
  height: 220px;
  overflow-y: auto;
  background: #fafafa;
  border: 1px solid #999999;
  padding: 8px;
  font-family: monospace;
  font-size: 14px;
  white-space: pre-line;
}
</style>
</head>

<body>

<div class="panel">

  <!-- NAGŁÓWEK -->
  <div class="header">
    <div class="title">SYSTEM PRESOSTATÓW</div>
    <div class="alarms" id="headerBar">AKTYWNE ALARMY: 0</div>
  </div>

  <!-- KAFELKI -->
  <div class="grid">
    <!-- generowane dynamicznie -->
    <script>
      for (let i = 1; i <= 44; i++) {
        document.write(
          '<div class="tile ok" id="p' + i + '">W-' + i + '</div>'
        );
      }
    </script>
  </div>

  <!-- HISTORIA -->
  <div class="history">
    <div class="history-title">HISTORIA ZDARZEŃ</div>
    <div class="history-box" id="historyBox"></div>
  </div>

</div>

<!-- ====== JS / WebSocket ====== -->
<script>
let socket;
let lastMsg = 0;

function connectWS() {
  socket = new WebSocket("ws://" + location.hostname + ":81/");

  socket.onmessage = function(event) {
    lastMsg = Date.now();
    let parts = event.data.split(";", 3);
    if (parts.length < 3) return;

    let st = parts[0];
    let alarms = parts[1];
    let hist = parts[2];

    for (let i = 0; i < 44; i++) {
      let t = document.getElementById("p" + (i+1));
      if (st[i] === "1") t.className = "tile alarm";
      else               t.className = "tile ok";
    }

    document.getElementById("headerBar").innerText =
      "AKTYWNE ALARMY: " + alarms;

    document.getElementById("historyBox").innerText = hist;
  };

  socket.onclose = () => location.reload();
  socket.onerror = () => location.reload();
}

connectWS();
</script>

</body>
</html>
