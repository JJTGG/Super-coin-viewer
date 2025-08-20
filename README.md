<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>🚀 Super Coin Viewer</title>
  <script src="https://unpkg.com/lightweight-charts/dist/lightweight-charts.standalone.production.js"></script>
  <style>
    body {
      margin: 0;
      background: #000;
      color: cyan;
      font-family: Arial, sans-serif;
      text-align: center;
    }
    h2 {
      margin: 10px 0;
    }
    #price {
      font-size: 22px;
      margin: 10px;
      color: gold;
    }
    #chart {
      height: 500px;
      width: 100%;
    }
    #controls {
      margin: 10px;
    }
    button {
      background: #111;
      color: cyan;
      border: 1px solid cyan;
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
