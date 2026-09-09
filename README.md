<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>BTC 백테스팅 대시보드</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body {
            font-family: 'Arial', sans-serif;
            background: #1a1a2e;
            color: #fff;
            min-height: 100vh;
            padding: 20px;
        }
        .container {
            max-width: 1400px;
            margin: 0 auto;
        }
        .header {
            text-align: center;
            margin-bottom: 30px;
        }
        .header h1 {
            font-size: 32px;
            margin-bottom: 10px;
        }
        .grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 20px;
            margin-bottom: 20px;
        }
        @media (max-width: 1024px) {
            .grid {
                grid-template-columns: 1fr;
            }
        }
        .panel {
            background: #16213e;
            border-radius: 8px;
            padding: 20px;
            border: 1px solid #0f3460;
        }
        .panel h2 {
            color: #00d4ff;
            margin-bottom: 15px;
            font-size: 18px;
            border-bottom: 2px solid #00d4ff;
            padding-bottom: 10px;
        }
        .settings {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
        }
        .setting-group {
            display: flex;
            flex-direction: column;
        }
        label {
            font-size: 12px;
            color: #aaa;
            margin-bottom: 5px;
            font-weight: bold;
        }
        input {
            padding: 10px;
            background: #0f3460;
            border: 1px solid #533483;
            border-radius: 4px;
            color: #fff;
            font-size: 14px;
        }
        input:focus {
            outline: none;
            border-color: #00d4ff;
            box-shadow: 0 0 5px #00d4ff;
        }
        textarea {
            width: 100%;
            height: 300px;
            padding: 15px;
            background: #0f3460;
            border: 1px solid #533483;
            border-radius: 4px;
            color: #64b5f6;
            font-family: 'Courier New', monospace;
            font-size: 13px;
            resize: vertical;
            margin-bottom: 10px;
        }
        textarea:focus {
            outline: none;
            border-color: #00d4ff;
        }
        .button-group {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            margin-top: 15px;
        }
        button {
            padding: 12px;
            background: #00d4ff;
            color: #000;
            border: none;
            border-radius: 4px;
            font-weight: bold;
            cursor: pointer;
            transition: 0.3s;
            font-size: 14px;
        }
        button:hover {
            background: #00b8cc;
        }
        button.danger {
            background: #e74c3c;
            color: #fff;
        }
        button.danger:hover {
            background: #c0392b;
        }
        .chart-box {
            position: relative;
            height: 400px;
        }
        .stats {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
            margin-top: 20px;
        }
        .stat {
            background: #0f3460;
            padding: 15px;
            border-radius: 4px;
            border-left: 4px solid #00d4ff;
            text-align: center;
        }
        .stat-label {
            font-size: 12px;
            color: #aaa;
            margin-bottom: 5px;
        }
        .stat-value {
            font-size: 24px;
            font-weight: bold;
            color: #00d4ff;
        }
        .stat.profit .stat-value {
            color: #2ecc71;
        }
        .stat.loss .stat-value {
            color: #e74c3c;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 15px;
            font-size: 13px;
        }
        table th {
            background: #0f3460;
            padding: 10px;
            text-align: left;
            color: #00d4ff;
            border-bottom: 2px solid #533483;
        }
        table td {
            padding: 10px;
            border-bottom: 1px solid #0f3460;
        }
        table tr:hover {
            background: #1a3a52;
        }
        .buy {
            color: #2ecc71;
            font-weight: bold;
        }
        .sell {
            color: #e74c3c;
            font-weight: bold;
        }
        .message {
            padding: 15px;
            border-radius: 4px;
            margin-top: 10px;
            display: none;
        }
        .message.show {
            display: block;
        }
        .error {
            background: #e74c3c;
            color: #fff;
        }
        .success {
            background: #2ecc71;
            color: #000;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>🚀 BTC 백테스팅 대시보드</h1>
            <p id="btcPrice" style="color: #00d4ff; font-size: 16px;">BTC: 로딩중...</p>
        </div>

        <div class="grid">
            <!-- 왼쪽: 설정 + 코드 -->
            <div>
                <div class="panel">
                    <h2>⚙️ 거래 설정</h2>
                    <div class="settings">
                        <div class="setting-group">
                            <label>초기 자본금 ($)</label>
                            <input type="number" id="initialCapital" value="10000" min="100">
                        </div>
                        <div class="setting-group">
                            <label>진입 자본금 ($)</label>
                            <input type="number" id="positionSize" value="1000" min="1">
                        </div>
                        <div class="setting-group">
                            <label>레버리지 배수</label>
                            <input type="number" id="leverage" value="1" min="1" max="100">
                        </div>
                        <div class="setting-group">
                            <label>거래 수수료 (%)</label>
                            <input type="number" id="fee" value="0.1" min="0" step="0.01">
                        </div>
                    </div>
                </div>

                <div class="panel" style="margin-top: 20px;">
                    <h2>📝 백테스팅 코드</h2>
                    <textarea id="strategyCode">import ta
import numpy as np

def strategy(data, prices):
    results = []
    
    # SMA 20 vs SMA 50
    sma_20 = ta.sma(prices, 20)
    sma_50 = ta.sma(prices, 50)
    
    for i in range(len(prices)):
        if i < 50:
            continue
        
        if sma_20[i] > sma_50[i] and sma_20[i-1] <= sma_50[i-1]:
            results.append({'type': 'BUY', 'price': prices[i], 'index': i})
        elif sma_20[i] < sma_50[i] and sma_20[i-1] >= sma_50[i-1]:
            results.append({'type': 'SELL', 'price': prices[i], 'index': i})
    
    return results</textarea>
                    
                    <div class="button-group">
                        <button onclick="runBacktest()">▶️ 백테스트 실행</button>
                        <button class="danger" onclick="resetAll()">🔄 초기화</button>
                    </div>
                    <div class="message error" id="errorMsg"></div>
                    <div class="message success" id="successMsg"></div>
                </div>
            </div>

            <!-- 오른쪽: 차트 + 결과 -->
            <div>
                <div class="panel">
                    <h2>📊 BTC 가격 차트</h2>
                    <div class="chart-box">
                        <canvas id="priceChart"></canvas>
                    </div>
                </div>

                <div class="panel" style="margin-top: 20px;">
                    <h2>💰 거래 통계</h2>
                    <div class="stats">
                        <div class="stat">
                            <div class="stat-label">현재 자산</div>
                            <div class="stat-value" id="currentBalance">$10,000</div>
                        </div>
                        <div class="stat profit">
                            <div class="stat-label">총 수익</div>
                            <div class="stat-value" id="totalProfit">$0</div>
                        </div>
                        <div class="stat">
                            <div class="stat-label">수익률</div>
                            <div class="stat-value" id="profitRate">0%</div>
                        </div>
                        <div class="stat">
                            <div class="stat-label">거래 횟수</div>
                            <div class="stat-value" id="tradeCount">0</div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- 거래 기록 -->
        <div class="panel">
            <h2>📋 거래 기록</h2>
            <table>
                <thead>
                    <tr>
                        <th>시간</th>
                        <th>타입</th>
                        <th>가격</th>
                        <th>수량</th>
                        <th>손익</th>
                    </tr>
                </thead>
                <tbody id="tradesBody">
                    <tr>
                        <td colspan="5" style="text-align: center; color: #aaa;">거래 기록 없음</td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>

    <script>
        let chart = null;
        let btcPrices = [];
        let trades = [];
        let balance = 10000;
        let btcHolding = 0;

        async function fetchBTCData() {
            try {
                const response = await axios.get('https://api.coingecko.com/api/v3/coins/bitcoin/market_chart', {
                    params: {
                        vs_currency: 'usd',
                        days: '30',
                        interval: 'daily'
                    }
                });
                btcPrices = response.data.prices.map(p => p[1]);
                updateBTCPrice();
                initChart();
                showSuccess('BTC 데이터 로드 완료!');
            } catch (error) {
                showError('BTC 데이터를 불러올 수 없습니다.');
            }
        }

        async function updateBTCPrice() {
            try {
                const response = await axios.get('https://api.coingecko.com/api/v3/simple/price', {
                    params: {
                        ids: 'bitcoin',
                        vs_currencies: 'usd'
                    }
                });
                const price = response.data.bitcoin.usd;
                document.getElementById('btcPrice').textContent = `BTC: $${price.toLocaleString()}`;
            } catch (error) {
                console.error('가격 업데이트 실패');
            }
        }

        function initChart() {
            const ctx = document.getElementById('priceChart').getContext('2d');
            chart = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: btcPrices.map((_, i) => `Day ${i + 1}`),
                    datasets: [
                        {
                            label: 'BTC 가격',
                            data: btcPrices,
                            borderColor: '#00d4ff',
                            backgroundColor: 'rgba(0, 212, 255, 0.1)',
                            tension: 0.4,
                            borderWidth: 2,
                            fill: true,
                            pointRadius: 0
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: {
                            labels: { color: '#aaa' }
                        }
                    },
                    scales: {
                        y: {
                            ticks: { color: '#aaa' },
                            grid: { color: '#0f3460' }
                        },
                        x: {
                            ticks: { color: '#aaa' },
                            grid: { color: '#0f3460' }
                        }
                    }
                }
            });
        }

        function calculateSMA(prices, period) {
            const sma = [];
            for (let i = 0; i < prices.length; i++) {
                if (i < period - 1) {
                    sma.push(null);
                } else {
                    const sum = prices.slice(i - period + 1, i + 1).reduce((a, b) => a + b, 0);
                    sma.push(sum / period);
                }
            }
            return sma;
        }

        function runBacktest() {
            try {
                if (btcPrices.length === 0) {
                    showError('BTC 데이터가 로드되지 않았습니다.');
                    return;
                }

                const initialCap = parseFloat(document.getElementById('initialCapital').value);
                const posSize = parseFloat(document.getElementById('positionSize').value);
                const lev = parseFloat(document.getElementById('leverage').value);
                const feeRate = parseFloat(document.getElementById('fee').value) / 100;

                balance = initialCap;
                btcHolding = 0;
                trades = [];

                const sma20 = calculateSMA(btcPrices, 20);
                const sma50 = calculateSMA(btcPrices, 50);

                for (let i = 1; i < btcPrices.length; i++) {
                    if (i < 50) continue;

                    const prevSignal = sma20[i - 1] > sma50[i - 1];
                    const currSignal = sma20[i] > sma50[i];
                    const price = btcPrices[i];

                    if (!prevSignal && currSignal && balance > 0) {
                        const amount = Math.min(posSize * lev, balance);
                        const btc = amount / price;
                        const fee = amount * feeRate;
                        balance -= (amount + fee);
                        btcHolding += btc;
                        trades.push({
                            time: `Day ${i + 1}`,
                            type: 'BUY',
                            price: price.toFixed(2),
                            amount: btc.toFixed(6),
                            pnl: '-' + fee.toFixed(2)
                        });
                    } else if (prevSignal && !currSignal && btcHolding > 0) {
                        const revenue = btcHolding * price;
                        const fee = revenue * feeRate;
                        balance += (revenue - fee);
                        trades.push({
                            time: `Day ${i + 1}`,
                            type: 'SELL',
                            price: price.toFixed(2),
                            amount: btcHolding.toFixed(6),
                            pnl: (revenue - fee).toFixed(2)
                        });
                        btcHolding = 0;
                    }
                }

                if (btcHolding > 0) {
                    const finalPrice = btcPrices[btcPrices.length - 1];
                    const revenue = btcHolding * finalPrice;
                    const fee = revenue * feeRate;
                    balance += (revenue - fee);
                    btcHolding = 0;
                }

                updateStats();
                updateTradesTable();
                updateChartTrades();
                showSuccess('백테스트 완료!');
            } catch (error) {
                showError('백테스트 실행 중 오류: ' + error.message);
            }
        }

        function updateStats() {
            const initialCap = parseFloat(document.getElementById('initialCapital').value);
            const profit = balance - initialCap;
            const profitRate = (profit / initialCap * 100).toFixed(2);
            
            document.getElementById('currentBalance').textContent = `$${balance.toFixed(2)}`;
            document.getElementById('totalProfit').textContent = `$${profit.toFixed(2)}`;
            document.getElementById('profitRate').textContent = `${profitRate}%`;
            document.getElementById('tradeCount').textContent = trades.length;
        }

        function updateTradesTable() {
            const tbody = document.getElementById('tradesBody');
            if (trades.length === 0) {
                tbody.innerHTML = '<tr><td colspan="5" style="text-align: center; color: #aaa;">거래 기록 없음</td></tr>';
                return;
            }
            tbody.innerHTML = trades.map(t => `
                <tr>
                    <td>${t.time}</td>
                    <td class="${t.type === 'BUY' ? 'buy' : 'sell'}">${t.type}</td>
                    <td>$${t.price}</td>
                    <td>${t.amount} BTC</td>
                    <td>${t.pnl}</td>
                </tr>
            `).join('');
        }

        function updateChartTrades() {
            const buys = trades.filter(t => t.type === 'BUY').map(t => {
                const day = parseInt(t.time.split(' ')[1]);
                return { x: day - 1, y: parseFloat(t.price) };
            });
            const sells = trades.filter(t => t.type === 'SELL').map(t => {
                const day = parseInt(t.time.split(' ')[1]);
                return { x: day - 1, y: parseFloat(t.price) };
            });

            if (chart) {
                chart.data.datasets = [{
                    label: 'BTC 가격',
                    data: btcPrices,
                    borderColor: '#00d4ff',
                    backgroundColor: 'rgba(0, 212, 255, 0.1)',
                    tension: 0.4,
                    borderWidth: 2,
                    fill: true,
                    pointRadius: 0,
                    showLine: true
                }];
                
                if (buys.length > 0) {
                    chart.data.datasets.push({
                        label: '매수',
                        data: buys,
                        borderColor: '#2ecc71',
                        backgroundColor: '#2ecc71',
                        pointRadius: 8,
                        showLine: false,
                        type: 'scatter'
                    });
                }
                
                if (sells.length > 0) {
                    chart.data.datasets.push({
                        label: '매도',
                        data: sells,
                        borderColor: '#e74c3c',
                        backgroundColor: '#e74c3c',
                        pointRadius: 8,
                        showLine: false,
                        type: 'scatter'
                    });
                }
                
                chart.update();
            }
        }

        function resetAll() {
            balance = parseFloat(document.getElementById('initialCapital').value);
            btcHolding = 0;
            trades = [];
            document.getElementById('tradesBody').innerHTML = '<tr><td colspan="5" style="text-align: center; color: #aaa;">거래 기록 없음</td></tr>';
            document.getElementById('currentBalance').textContent = `$${balance.toFixed(2)}`;
            document.getElementById('totalProfit').textContent = '$0';
            document.getElementById('profitRate').textContent = '0%';
            document.getElementById('tradeCount').textContent = '0';
            initChart();
            showSuccess('초기화 완료!');
        }

        function showError(msg) {
            const el = document.getElementById('errorMsg');
            el.textContent = msg;
            el.classList.add('show');
            setTimeout(() => el.classList.remove('show'), 3000);
        }

        function showSuccess(msg) {
            const el = document.getElementById('successMsg');
            el.textContent = msg;
            el.classList.add('show');
            setTimeout(() => el.classList.remove('show'), 3000);
        }

        window.addEventListener('load', () => {
            fetchBTCData();
            setInterval(updateBTCPrice, 60000);
        });
    </script>
</body>
</html>