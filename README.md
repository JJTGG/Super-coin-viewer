<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>🚀 Super Coin Viewer</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #0d0d0d;
      color: #fff;
      text-align: center;
      margin: 0;
      padding: 0;
    }
    .container {
      max-width: 950px;
      margin: 20px auto;
      padding: 20px;
      background: #111;
      border-radius: 15px;
      box-shadow: 0 0 25px cyan;
    }
    h1 { color: cyan; }
    .price { font-size: 2em; margin: 10px 0; }
    .change { font-size: 1.2em; }
    .negative { color: red; }
    .positive { color: lime; }
    button, select {
      background: cyan;
      border: none;
      padding: 10px 20px;
      border-radius: 8px;
      cursor: pointer;
      margin: 5px;
    }
    button:hover, select:hover {
      background: magenta;
      color: #fff;
    }
    .logs {
      text-align: left;
      margin-top: 20px;
      background: #000;
      padding: 10px;
      border-radius: 10px;
      max-height: 200px;
      overflow-y: auto;
      font-size: 0.9em;
    }
    .chart-container {
      margin-top: 20px;
      height: 400px;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>🚀 Super Coin Viewer</h1>
    <div class="price">Price: $<span id="price">0.00</span></div>
    <div class="change">24h Change: <span id="change">0%</span></div>
    <button onclick="fetchPrice()">Refresh Now</button>
    <button onclick="exportLogs()">Export Logs (CSV)</button>

    <div id="chart" class="chart-container"></div>
    <div class="logs" id="logs"></div>
  </div>

  <!-- Chart.js -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <!-- Notification sound -->
  <audio id="notifSound" src="https://cdn.wapka.org/00gas2/780a16d90e78e425791a8d741fe9572f/mixkit-epic-impact-afar-explosion-2782.wav"></audio>

  <script>
    let logs = [];
    let chart;
    let chartData = [];
    let chartLabels = [];

    async function fetchPrice() {
      let price = 0, change = 0, source = "Binance";

      try {
        // Binance API
        const res = await fetch("https://api.binance.com/api/v3/ticker/24hr?symbol=BTCUSDT");
        const data = await res.json();
        price = parseFloat(data.lastPrice);
        change = parseFloat(data.priceChangePercent);
      } catch (err) {
        console.warn("Binance failed, trying LBank...");
        try {
          // LBank API
          const res = await fetch("https://api.lbank.info/v2/ticker.do?symbol=btc_usdt");
          const data = await res.json();
          price = parseFloat(data.data[0].ticker.latest);
          change = parseFloat(data.data[0].ticker.change);
          source = "LBank";
        } catch (err2) {
          console.error("Both Binance & LBank failed!");
          logRequest("ERR", "-");
          return;
        }
      }

      // Update UI
      document.getElementById("price").innerText = price.toFixed(2);
      document.getElementById("change").innerText = change.toFixed(2) + "%";
      const changeEl = document.getElementById("change");
      changeEl.className = change >= 0 ? "positive" : "negative";

      // Update Chart
      const now = new Date().toLocaleTimeString();
      chartLabels.push(now);
      chartData.push(price);
      if (chartLabels.length > 20) {
        chartLabels.shift();
        chartData.shift();
      }
      chart.update();

      // Log request
      logRequest(source, price);

      // Notification alert
      if (Notification.permission === "granted") {
        new Notification("BTC Update", { body: `Price: $${price}`, icon: "https://cryptologos.cc/logos/bitcoin-btc-logo.png" });
        document.getElementById("notifSound").play();
      }
    }

    function logRequest(source, price) {
      const time = new Date().toLocaleTimeString();
      const log = `${time} | ${source} | ${price}`;
      logs.unshift(log);
      document.getElementById("logs").innerHTML = logs.slice(0, 20).join("<br>");
    }

    function exportLogs() {
      let csvContent = "data:text/csv;charset=utf-8,Time,Source,Price\n";
      logs.forEach(log => {
        csvContent += log.replace(/ \| /g, ",") + "\n";
      });
      const encodedUri = encodeURI(csvContent);
      const link = document.createElement("a");
      link.setAttribute("href", encodedUri);
      link.setAttribute("download", "logs.csv");
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    }

    function initChart() {
      const ctx = document.getElementById("chart").getContext("2d");
      chart = new Chart(ctx, {
        type: "line",
        data: {
          labels: chartLabels,
          datasets: [{
            label: "BTC Price (USD)",
            data: chartData,
            borderColor: "cyan",
            backgroundColor: "rgba(0,255,255,0.2)"
          }]
        },
        options: {
          responsive: true,
          scales: {
            x: { ticks: { color: "#fff" } },
            y: { ticks: { color: "#fff" } }
          }
        }
      });
    }

    // Ask notification permission
    if (Notification.permission !== "granted") {
      Notification.requestPermission();
    }

    // Init
    window.onload = () => {
      const chartCanvas = document.createElement("canvas");
      document.getElementById("chart").appendChild(chartCanvas);
      initChart();
      fetchPrice();
      setInterval(fetchPrice, 10000);
    };
  </script>
</body>
</html>      border: 1px solid cyan;
      padding: 5px 10px;
      margin: 3px;
      border-radius: 5px;
      cursor: pointer;
    }
    button:hover {
      background: cyan;
      color: black;
    }
  </style>
</head>
<body>
  <h2>🚀 Super Coin Viewer</h2>
  <div id="price">Loading price...</div>
  <div id="controls">
    <button onclick="switchChart('line')">📈 Line</button>
    <button onclick="switchChart('candle')">🕯️ Candlestick</button>
  </div>
  <div id="chart"></div>

  <!-- Notification sound -->
  <audio id="alertSound" src="https://cdn.wapka.org/00gas2/780a16d90e78e425791a8d741fe9572f/mixkit-epic-impact-afar-explosion-2782.wav"></audio>

  <script>
    // Create chart
    const chart = LightweightCharts.createChart(document.getElementById('chart'), {
      layout: { background: { color: '#000' }, textColor: 'cyan' },
      grid: { vertLines: { color: '#222' }, horzLines: { color: '#222' } },
    });

    const candleSeries = chart.addCandlestickSeries();
    const lineSeries = chart.addLineSeries({ color: 'cyan' });

    let activeSeries = lineSeries;
    let lastPrice = null;

    function switchChart(type) {
      if (type === 'line') {
        candleSeries.applyOptions({ visible: false });
        lineSeries.applyOptions({ visible: true });
        activeSeries = lineSeries;
      } else {
        lineSeries.applyOptions({ visible: false });
        candleSeries.applyOptions({ visible: true });
        activeSeries = candleSeries;
      }
    }

    // Fetch candlestick data from Binance
    async function fetchCandles() {
      try {
        const res = await fetch('https://api.binance.com/api/v3/klines?symbol=BTCUSDT&interval=1m&limit=50');
        const data = await res.json();
        const candles = data.map(c => ({
          time: c[0] / 1000,
          open: parseFloat(c[1]),
          high: parseFloat(c[2]),
          low: parseFloat(c[3]),
          close: parseFloat(c[4]),
        }));
        candleSeries.setData(candles);
        lineSeries.setData(candles.map(c => ({ time: c.time, value: c.close })));
      } catch (e) {
        console.error("Binance API error", e);
      }
    }

    // Fetch live price from LBank
    async function fetchPrice() {
      try {
        const res = await fetch('https://api.lbkex.com/v2/ticker.do?symbol=btc_usdt');
        const json = await res.json();
        const price = parseFloat(json.data[0].ticker.latest);
        document.getElementById('price').innerText = `₿ BTC Price: $${price}`;

        // 🔔 Alert if price changes significantly
        if (lastPrice !== null && Math.abs(price - lastPrice) > 50) { // change threshold $50
          triggerAlert(price);
        }
        lastPrice = price;
      } catch (e) {
        document.getElementById('price').innerText = '⚠️ Price fetch failed';
      }
    }

    function triggerAlert(price) {
      // Play sound
      document.getElementById('alertSound').play();

      // Send browser notification
      if (Notification.permission === "granted") {
        new Notification("🚨 BTC Price Alert!", {
          body: `BTC moved to $${price}`,
          icon: "https://cryptologos.cc/logos/bitcoin-btc-logo.png",
        });
      }
    }

    // Ask for notification permission
    if (Notification.permission !== "granted") {
      Notification.requestPermission();
    }

    // Initial load
    fetchCandles();
    fetchPrice();

    // Update every 10s for price, 1m for candles
    setInterval(fetchPrice, 10000);
    setInterval(fetchCandles, 60000);
  </script>
</body>
</html>
