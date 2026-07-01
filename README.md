
<html lang="id">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>kipas ruang operator</title>
  <script src="https://unpkg.com/mqtt/dist/mqtt.min.js"></script>
  <style>
    body {
      font-family: Arial, Helvetica, sans-serif;
      background: #2a9b90;
      background: linear-gradient(90deg, rgba(42, 155, 144, 0.36) 0%, rgba(87, 87, 199, 0.96) 96%, rgba(237, 221, 83, 1) 100%);
      color: #222;
      margin: 20px;
    }
    h2 { text-align: center; margin-bottom: 20px; }
    .card {
      background: #fff;;
      border-radius: 12px;
      box-shadow: 0 2px 6px rgba(0,0,0,0.08);
      padding: 15px;
      margin-bottom: 15px;
    }
    button {
      padding: 8px 14px;
      margin: 5px;
      border-radius: 8px;
      border: none;
      cursor: pointer;
      transition: 0.2s;
    }
    button:hover { opacity: 0.85; }
    .speed1 { background: #2ecc71; color: #fff; }
    .off { background: #e74c3c; color: #fff; }
    .speed2 { background: #2ecc71; color: #fff; }
    .speed3 { background: #2ecc71; color: #fff; }
    .kanan { background: #2ecc71; color: #fff; }
    .kiri { background: #2ecc71; color: #fff; }
    .stop { background: #e74c3c;  color: #fff; }
    .auto { background: #2ecc71; color: #fff; }

    .tele{max-height : 300px;
          overflow -y : auto;}
      
    #status { font-weight: bold; }
    .grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
      text-align: center;
    }
    
  </style>
</head>

<body>
  <h2>KIPAS RUANG OPERATOR</h2>

  <div class="card">
    <strong>Koneksi</strong><br>
    Broker: <code>wss://broker.hivemq.com:8884/mqtt</code><br>
    Client ID: <span id="cid">...</span><br>
    Status: <span id="status">disconnected</span>
  </div>

  <div class="card">
    <strong>KONTROL</strong><br>
    <button id="off" class="off">off</button>
    <button id="speed1" class="speed1">speed1</button>
    <button id="speed2" class="speed2">speed2</button>
    <button id="speed3" class="speed3">speed3</button><br>
    <button id="kanan" class="kanan">noleh kanan</button>
    <button id="kiri" class="kiri">noleh kiri</button><br>
    <button id="stop" class="stop">stop</button><br>
    <button id="auto" class="auto">tolah toleh</button>
  </div>


  <div class="card">
    <strong>Data Telemetri</strong>
    <div class="tele" id="tele">Menunggu data...</div>
  </div>

<script>
  // ====== MQTT HiveMQ Secure ======
  const brokerUrl = "wss://broker.emqx.io:8084/mqtt";
  const cid = "web-" + Math.random().toString(16).substr(2, 8);
  document.getElementById("cid").textContent = cid;

  const client = mqtt.connect(brokerUrl, { clientId: cid });

  const statusEl = document.getElementById("status");
  const teleEl = document.getElementById("tele");

 
  client.on("connect", () => {
    statusEl.textContent = "connected";
    statusEl.style.color = "green";
    client.subscribe("esp8266/tele/#");
    client.subscribe("esp8266/ack/#");
  });

  client.on("reconnect", () => {
    statusEl.textContent = "reconnecting...";
    statusEl.style.color = "orange";
  });
  client.on("close", () => {
    statusEl.textContent = "disconnected";
    statusEl.style.color = "red";
  });
  client.on("error", (err) => {
    console.error("MQTT Error:", err);
    statusEl.textContent = "error";
    statusEl.style.color = "red";
  });

  // ====== Menerima pesan ======
  client.on("message", (topic, payload) => {
    const msg = payload.toString();
    const time = new Date().toLocaleTimeString();
    teleEl.innerHTML = `<div>[${time}] ${topic} → ${msg}</div>` + teleEl.innerHTML;
    

    // DC & CH
    
    
  });


  // ====== Fungsi publish kontrol ======
  function publishCmd(sub, payload) {
    client.publish("esp8266/cmd/" + sub, payload.toString());
  }

  // ====== Tombol kontrol ======
  document.getElementById("speed1").addEventListener("click", () => publishCmd("status_relay", "1"));
  document.getElementById("off").addEventListener("click", () => publishCmd("status_relay", "0"));
  document.getElementById("speed2").addEventListener("click", () => publishCmd("status_relay", "2"));
  document.getElementById("speed3").addEventListener("click", () => publishCmd("status_relay", "3"));
  document.getElementById("kanan").addEventListener("click", () => publishCmd("my_servo", 1));
  document.getElementById("kiri").addEventListener("click", () => publishCmd("my_servo", 2));
  document.getElementById("stop").addEventListener("click", () => publishCmd("my_servo", 0));
  document.getElementById("auto").addEventListener("click", () => publishCmd("my_servo", 3));
</script>
</body>
</html>
