<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Robô Forex Brasil - Painel de Análise</title>
    <!-- 
      BIBLIOTECAS:
      1. Lightweight Charts: Para o gráfico de velas.
      2. Technical Indicators: Para calcular RSI, MACD, BBands, etc.
      3. Tone.js: Para gerar sons de alerta.
    -->
    <script src="https://cdn.jsdelivr.net/npm/lightweight-charts@4.1.0/dist/lightweight-charts.standalone.production.js"></script>
    <script src="https://unpkg.com/technicalindicators@3.1.0/dist/browser.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.js"></script>
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@700;900&family=Roboto:wght@400;700&display=swap');
        
        :root {
            --primary-green: #00ff00;
            --dark-green: #003300;
            --bg-dark: #001a00;
            --bg-semi-transparent: rgba(0, 30, 0, 0.8);
            --glow-color: rgba(0, 255, 0, 0.7);
            --buy-color: #006400;
            --sell-color: #8B0000;
            --buy-glow: rgba(0, 255, 0, 0.8);
            --sell-glow: rgba(255, 0, 0, 0.8);
            --error-color: #800;
            --error-glow: rgba(255, 0, 0, 0.5);
        }

        body {
            margin: 0;
            padding: 0;
            background: linear-gradient(135deg, #000, var(--bg-dark), #000);
            font-family: 'Roboto', sans-serif;
            color: #fff;
            overflow-x: hidden;
        }

        .header {
            background: linear-gradient(45deg, #000, var(--dark-green), #000);
            padding: 20px;
            text-align: center;
            border-bottom: 2px solid var(--primary-green);
            box-shadow: 0 0 30px var(--glow-color);
            position: relative;
        }

        .header h1 {
            font-family: 'Orbitron', sans-serif;
            font-size: clamp(2em, 7vw, 3.5em); /* Responsivo */
            margin: 0;
            color: var(--primary-green);
            text-shadow: 0 0 15px var(--primary-green), 0 0 30px var(--primary-green);
            animation: glow 2s infinite alternate;
        }

        .bandeira {
            position: absolute;
            top: 50%;
            right: 20px;
            transform: translateY(-50%);
            font-size: clamp(2em, 10vw, 3em);
            text-shadow: 0 0 10px #ff0; /* Sombra amarela para a bandeira */
        }

        .container {
            max-width: 1200px;
            margin: 20px auto;
            padding: 20px;
            background: var(--bg-semi-transparent);
            border: 2px solid var(--primary-green);
            border-radius: 20px;
            box-shadow: 0 0 40px var(--glow-color) inset;
        }

        .controls {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 15px;
            margin-bottom: 20px;
            padding: 20px;
            background: rgba(0, 50, 0, 0.5);
            border-radius: 15px;
        }

        select, button, .btn-whatsapp {
            padding: 15px;
            font-size: 16px;
            border-radius: 10px;
            border: 1px solid var(--primary-green);
            background: var(--bg-dark);
            color: var(--primary-green);
            font-weight: bold;
            font-family: 'Roboto', sans-serif;
            cursor: pointer;
            transition: all 0.3s ease;
            width: 100%;
            box-sizing: border-box; /* Garante que padding não afete a largura */
            text-align: center;
        }

        button, .btn-whatsapp {
            background: linear-gradient(45deg, var(--dark-green), #006600);
            box-shadow: 0 0 10px var(--glow-color);
        }

        button:hover, .btn-whatsapp:hover {
            transform: scale(1.05);
            box-shadow: 0 0 25px var(--glow-color);
            background: linear-gradient(45deg, #006600, var(--dark-green));
            color: #fff;
        }

        .btn-whatsapp {
            background: #25D366; /* Cor do WhatsApp */
            color: white;
            text-decoration: none;
            border-color: #fff;
            font-weight: 700;
        }
        .btn-whatsapp:hover {
            background: #128C7E;
            box-shadow: 0 0 20px #25D366;
        }

        .chart-container {
            height: 450px;
            margin: 20px 0;
            border: 2px solid var(--primary-green);
            border-radius: 15px;
            overflow: hidden;
            box-shadow: 0 0 20px var(--glow-color);
        }

        #resultado {
            font-size: clamp(2.5em, 10vw, 4.5em); /* Responsivo */
            text-align: center;
            padding: 30px;
            border-radius: 20px;
            margin: 20px 0;
            animation: pulse 2s infinite;
            font-family: 'Orbitron', sans-serif;
            text-shadow: 0 0 20px;
            transition: all 0.5s ease;
        }

        .buy {
            background: var(--buy-color);
            box-shadow: 0 0 60px var(--buy-glow);
            text-shadow: 0 0 20px var(--buy-glow);
            color: #fff;
        }

        .sell {
            background: var(--sell-color);
            box-shadow: 0 0 60px var(--sell-glow);
            text-shadow: 0 0 20px var(--sell-glow);
            color: #fff;
        }

        .neutro {
            background: #333;
            color: #aaa;
        }
        
        .erro {
            background: var(--error-color);
            color: #f88;
            box-shadow: 0 0 50px var(--error-glow);
            font-size: clamp(1.5em, 7vw, 2.5em); /* Menor para caber o texto */
        }

        .info {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
            gap: 15px;
            margin: 20px 0;
        }

        .card {
            background: rgba(0, 100, 0, 0.3);
            padding: 20px;
            border-radius: 15px;
            border: 1px solid var(--primary-green);
            text-align: center;
        }
        
        .card strong {
            color: var(--primary-green);
            font-size: 1.1em;
            display: block;
            margin-bottom: 10px;
        }

        .card span {
            font-size: 1.5em;
            font-weight: 700;
            font-family: 'Orbitron', sans-serif;
            color: #fff;
        }

        /* Animações */
        @keyframes glow {
            from { text-shadow: 0 0 15px var(--primary-green); }
            to { text-shadow: 0 0 30px var(--primary-green), 0 0 40px var(--primary-green); }
        }

        @keyframes pulse {
            0%, 100% { transform: scale(1); }
            50% { transform: scale(1.03); }
        }

    </style>
</head>
<body>
    <div class="header">
        <h1>PAINEL DE ANÁLISE FOREX</h1>
        <div class="bandeira">🇧🇷</div>
    </div>

    <div class="container">
        <div class="controls">
            <select id="par" title="Par de Moedas">
                <optgroup label="MAJORES">
                    <option value="EURUSD">EUR/USD</option>
                    <option value="GBPUSD">GBP/USD</option>
                    <option value="USDJPY">USD/JPY</option>
                    <option value="USDCAD">USD/CAD</option>
                    <option value="AUDUSD">AUD/USD</option>
                    <option value="NZDUSD">NZD/USD</option>
                </optgroup>
                <optgroup label="EXÓTICOS BR">
                    <option value="USDBRL">USD/BRL</option>
                    <option value="EURBRL">EUR/BRL</option>
                </optgroup>
                <optgroup label="CRUZADOS">
                    <option value="GBPJPY">GBP/JPY</option>
                    <option value="EURJPY">EUR/JPY</option>
                    <option value="EURGBP">EUR/GBP</option>
                </optgroup>
            </select>

            <select id="timeframe" title="Timeframe">
                <option value="1">1 minuto (Scalping)</option>
                <option value="5" selected>5 minutos (Padrão)</option>
                <option value="15">15 minutos</option>
                <option value="30">30 minutos</option>
                <option value="60">1 hora</option>
            </select>

            <select id="estrategia" title="Estratégia de Análise">
                <option value="1">RSI + MACD + SMA (Padrão)</option>
                <option value="2">Bandas de Bollinger</option>
                <option value="3">Cruzamento EMA 9 x 21</option>
                <option value="4">SuperTrend Pro</option>
                <option value="5">Ichimoku Cloud</option>
            </select>

            <button onclick="iniciar()">INICIAR ANÁLISE</button>
            <a href="#" class="btn-whatsapp" id="whatsappBtn" target="_blank">Abrir no WhatsApp</a>
        </div>

        <div class="chart-container" id="chart"></div>

        <div id="resultado" class="neutro">AGUARDANDO SINAL...</div>

        <div class="info">
            <div class="card"><strong>Preço Atual</strong><br><span id="preco">-</span></div>
            <div class="card"><strong>Indicador</strong><br><span id="indicador-nome">-</span></div>
            <div class="card"><strong>Valor</strong><br><span id="indicador-valor">-</span></div>
            <div class="card"><strong>Força</strong><br><span id="forca">-</span></div>
            <div class="card"><strong>Horário (BR)</strong><br><span id="hora">-</span></div>
        </div>
    </div>
    
    <script>
        // *** INTERRUPTOR MESTRE ***
        // Este é o modo "REAL". Ele tentará buscar dados reais.
        // Se falhar (como no seu preview), ele entrará em modo de simulação.
        const FORCE_SIMULATION_MODE = false;

        const API_KEY = 'd46lg8hr01qgc9etdjd0d46lg8hr01qgc9etdjdg'; 
        let chart, series;
        let updateInterval = null; 
        let ultimoSinal = '';
        
        let websocket = null;
        let simboloAtual = '';
        
        const ti = window.ti;

        const synthBuy = new Tone.Synth().toDestination();
        const synthSell = new Tone.Synth().toDestination();
        const synthForte = new Tone.Synth({ oscillator: { type: 'sine' } }).toDestination();

        function playBuySound() { synthBuy.triggerAttackRelease("C5", "8n", Tone.now()); }
        function playSellSound() { synthSell.triggerAttackRelease("G4", "8n",Tone.now()); }
        function playForteSound() { synthForte.triggerAttackRelease("G5", "16n", Tone.now()); }

        
        function criarGrafico() {
            if (chart) return true; 
            
            try { 
                const chartContainer = document.getElementById('chart');
                chart = LightweightCharts.createChart(chartContainer, {
                    width: chartContainer.clientWidth,
                    height: 450,
                    layout: { backgroundColor: '#000', textColor: '#00ff00' }, 
                    grid: { vertLines: { color: '#003300' }, horzLines: { color: '#003300' } },
                    crosshair: { mode: LightweightCharts.CrosshairMode.Normal },
                    rightPriceScale: { borderColor: '#00ff00' },
                    timeScale: { borderColor: '#00ff00', timeVisible: true, secondsVisible: true }
                });
                series = chart.addCandlestickSeries({
                    upColor: '#00ff00', downColor: '#ff0000',
                    borderUpColor: '#00ff00', wickUpColor: '#00ff00',
                    borderDownColor: '#ff0000', wickDownColor: '#ff0000'
                });
                
                window.addEventListener('resize', () => {
                    if (chart) {
                        chart.resize(chartContainer.clientWidth, 450);
                    }
                });
                
            } catch (e) {
                console.error("Erro ao criar o gráfico:", e);
                document.getElementById('resultado').innerHTML = 'ERRO AO CRIAR GRÁFICO';
                document.getElementById('resultado').className = 'erro';
                return false;
            }
            return true;
        }

        function iniciar() {
            Tone.start().catch(e => console.warn("Tone.js context could not start:", e));

            document.getElementById('resultado').innerHTML = 'ANALISANDO...';
            document.getElementById('resultado').className = 'neutro';

            if (updateInterval) {
                clearInterval(updateInterval);
            }

            buscarDados(true); 
            
            if (!FORCE_SIMULATION_MODE) {
                conectarWebSocket();
            }
            
            if (!FORCE_SIMULATION_MODE) {
                updateInterval = setInterval(() => {
                    buscarDados(true); 
                }, 30000); 
            }
        }
        
        function conectarWebSocket() {
            const par = document.getElementById('par').value;
            const novoSimbolo = `OANDA:${par}`;

            if (!websocket || websocket.readyState === WebSocket.CLOSED) {
                websocket = new WebSocket(`wss://ws.finnhub.io?token=${API_KEY}`);
                websocket.onopen = () => {
                    console.log("WebSocket CONECTADO");
                    document.getElementById('indicador-nome').innerText = "Tempo Real";
                    document.getElementById('indicador-valor').innerText = "Conectado";
                    websocket.send(JSON.stringify({'type':'subscribe', 'symbol': novoSimbolo}));
                    simboloAtual = novoSimbolo;
                };
                websocket.onmessage = (event) => {
                    const data = JSON.parse(event.data);
                    if (data.type === 'trade' && data.data) {
                        const trade = data.data[0];
                        const preco = trade.p;
                        const timestamp = trade.t / 1000;
                        document.getElementById('preco').innerText = preco.toFixed(5);
                        series.update({ time: timestamp, close: preco });
                    }
                };
                websocket.onerror = (error) => console.error("WebSocket Erro: ", error);
                websocket.onclose = () => {
                    console.warn("WebSocket DESCONECTADO.");
                    document.getElementById('indicador-nome').innerText = "Tempo Real";
                    document.getElementById('indicador-valor').innerText = "Desconectado";
                };
            } 
            else if (websocket.readyState === WebSocket.OPEN && simboloAtual !== novoSimbolo) {
                websocket.send(JSON.stringify({'type':'unsubscribe', 'symbol': simboloAtual}));
                websocket.send(JSON.stringify({'type':'subscribe', 'symbol': novoSimbolo}));
                simboloAtual = novoSimbolo;
            }
        }


        function parsePair(par) {
            const base = par.substring(0, 3);
            const symbol = par.substring(3, 6);
            return { base, symbol };
        }

        async function buscarDados(runAnalysis = false) { 
            const par = document.getElementById('par').value;
            const tf = document.getElementById('timeframe').value;
            
            document.getElementById('hora').innerText = new Date().toLocaleTimeString('pt-BR');
            
            if (FORCE_SIMULATION_MODE) {
                carregarGraficoSimulado(runAnalysis);
                return; 
            }

            try {
                // 1. Tenta Finnhub
                const to = Math.floor(Date.now() / 1000);
                const from = to - (150 * 60 * parseInt(tf)); 
                const url = `https://finnhub.io/api/v1/forex/candle?symbol=OANDA:${par}&resolution=${tf}&from=${from}&to=${to}&token=${API_KEY}`;
                
                const res = await fetch(url);
                if (!res.ok) throw new Error(`Finnhub status: ${res.status}`); 
                
                const data = await res.json();

                if (data.s === 'ok' && data.c && data.c.length > 0) {
                    const candles = data.t.map((t, i) => ({
                        time: t,
                        open: data.o[i],
                        high: data.h[i],
                        low: data.l[i],
                        close: data.c[i]
                    }));
                    series.setData(candles);
                    setTimeout(() => chart.timeScale().fitContent(), 100); 
                    
                    if (runAnalysis) {
                        analisarTecnico(data, par); 
                    } else {
                        document.getElementById('preco').innerText = data.c[data.c.length - 1].toFixed(5);
                    }
                } else {
                    throw new Error('Finnhub OK, mas sem dados.');
                }
            } catch (error) {
                console.warn('Falha ao buscar dados da Finnhub (HTTP). Usando fallback.', error.message);
                
                // 2. Fallback: open.er-api.com
                try {
                    const { base, symbol } = parsePair(par);
                    const fallbackUrl = `https://open.er-api.com/v6/latest/${base}`;
                    const res = await fetch(fallbackUrl);
                    if (!res.ok) throw new Error(`Fallback status: ${res.status}`);
                    const rates = await res.json();
                    
                    if (rates.rates && rates.rates[symbol]) {
                        const preco = rates.rates[symbol].toFixed(5);
                        document.getElementById('preco').innerText = preco;
                        
                        if (runAnalysis) {
                            document.getElementById('resultado').innerHTML = 'USANDO BACKUP'; 
                            document.getElementById('indicador-nome').innerText = 'Fallback';
                            document.getElementById('indicador-valor').innerText = 'Ativo';
                        }
                    } else {
                        throw new Error(`Taxa '${symbol}' não encontrada no fallback.`);
                    }

                } catch (fallbackError) {
                    // 3. MODO SIMULAÇÃO: Ambas as APIs falharam
                    console.error('API Fallback também falhou. Entrando em modo de simulação.', fallbackError.message);
                    carregarGraficoSimulado(runAnalysis); // Chama a função de simulação
                }
            }
        }
        
        function carregarGraficoSimulado(runAnalysis = false) {
            const par = document.getElementById('par').value;
            
            document.getElementById('resultado').innerHTML = runAnalysis ? 'ANALISANDO (SIMULAÇÃO)' : 'MODO SIMULAÇÃO ATIVO';
            document.getElementById('resultado').className = runAnalysis ? 'neutro' : 'erro';
            document.getElementById('indicador-nome').innerText = 'Simulação';
            document.getElementById('indicador-valor').innerText = 'Ativo';
            
            if(updateInterval) clearInterval(updateInterval); 
            if(websocket) websocket.close(); 
            
            console.log("Carregando dados simulados..."); 
            const simData = gerarDadosSimulados(par);
            series.setData(simData.candles);
            setTimeout(() => chart.timeScale().fitContent(), 100); 
            
            if (runAnalysis) {
                analisarTecnico(simData.data, par);
            } else {
                document.getElementById('preco').innerText = simData.data.c[simData.data.c.length - 1].toFixed(5);
            }
        }


        function gerarDadosSimulados(par) {
            let preco = 1.0800; 
            if (par.includes('JPY')) preco = 150.00;
            if (par.includes('BRL')) preco = 5.20;

            const data = { o: [], h: [], l: [], c: [], t: [] };
            const candles = [];
            let timestamp = Math.floor(Date.now() / 1000) - (150 * 60 * 5); 

            for (let i = 0; i < 150; i++) {
                const open = preco;
                const close = open + (Math.random() - 0.5) * 0.005 * preco;
                const high = Math.max(open, close) + (Math.random() * 0.003 * preco);
                const low = Math.min(open, close) - (Math.random() * 0.003 * preco);
                
                data.o.push(open);
                data.h.push(high);
                data.l.push(low);
                data.c.push(close);
                data.t.push(timestamp);
                
                candles.push({ time: timestamp, open, high, low, close });
                
                preco = close;
                timestamp += (60 * 5); 
            }
            return { data, candles };
        }


        // **** IMPLEMENTAÇÃO DAS ESTRATÉGIAS DE ANÁLISE ****
        function analisarTecnico(data, par) {
            if (typeof ti === 'undefined') {
                console.error("Biblioteca TechnicalIndicators (ti) não carregou.");
                document.getElementById('resultado').innerHTML = 'ERRO DE BIBLIOTECA';
                document.getElementById('resultado').className = 'erro';
                return;
            }
            
            const { o: open, h: high, l: low, c: close } = data;
            
            if (!Array.isArray(close) || close.length === 0) {
                 document.getElementById('resultado').innerHTML = 'DADOS INVÁLIDOS';
                 document.getElementById('resultado').className = 'erro';
                 console.error("analisarTecnico recebeu dados inválidos:", data);
                 return;
            }
            
            const preco = close[close.length - 1];
            document.getElementById('preco').innerText = preco.toFixed(5); 

            const estrategia = document.getElementById('estrategia').value;
            
            let sinal = 'AGUARDAR';
            let cor = 'neutro';
            let forca = 'Neutra';
            let indicadorNome = '-';
            let indicadorValor = '-';

            try {
                if (close.length < 52) { 
                    throw new Error("Dados insuficientes para análise.");
                }

                switch (estrategia) {
                    // ESTRATÉGIA 1: RSI + MACD + SMA 50 (Padrão)
                    case '1': {
                        indicadorNome = 'RSI(14)';
                        const rsi = ti.RSI.calculate({ values: close, period: 14 }).pop();
                        indicadorValor = rsi.toFixed(1);
                        
                        const macd = ti.MACD.calculate({ values: close, fastPeriod: 12, slowPeriod: 26, signalPeriod: 9 }).pop();
                        const sma50 = ti.SMA.calculate({ values: close, period: 50 }).pop();

                        if (rsi < 30 && macd.histogram > 0 && preco > sma50) {
                            sinal = 'COMPRA FORTE'; cor = 'buy'; forca = 'FORTE';
                        } else if (rsi > 70 && macd.histogram < 0 && preco < sma50) {
                            sinal = 'VENDA FORTE'; cor = 'sell'; forca = 'FORTE';
                        } else if (rsi < 40 && macd.histogram > 0) {
                            sinal = 'COMPRA'; cor = 'buy'; forca = 'Média';
                        } else if (rsi > 60 && macd.histogram < 0) {
                            sinal = 'VENDA'; cor = 'sell'; forca = 'Média';
                        }
                        break;
                    }

                    // ESTRATÉGIA 2: Bandas de Bollinger
                    case '2': {
                        indicadorNome = 'Bollinger(20,2)';
                        const bbands = ti.BollingerBands.calculate({ values: close, period: 20, stdDev: 2 }).pop();
                        indicadorValor = `L: ${bbands.lower.toFixed(4)}`;

                        if (preco < bbands.lower) {
                            sinal = 'COMPRA'; cor = 'buy'; forca = 'Média';
                        } else if (preco < (bbands.lower * 0.998)) { 
                            sinal = 'COMPRA FORTE'; cor = 'buy'; forca = 'FORTE';
                        } else if (preco > bbands.upper) {
                            sinal = 'VENDA'; cor = 'sell'; forca = 'Média';
                        } else if (preco > (bbands.upper * 1.002)) { 
                            sinal = 'VENDA FORTE'; cor = 'sell'; forca = 'FORTE';
                        }
                        break;
                    }

                    // ESTRATÉGIA 3: Cruzamento EMA 9 x 21
                    case '3': {
                        indicadorNome = 'EMA(9) x EMA(21)';
                        const emas = ti.EMA.calculate({ values: close, period: 9 });
                        const emal = ti.EMA.calculate({ values: close, period: 21 });
                        const ema9 = emas.pop();
                        const ema21 = emal.pop();
                        const prevEma9 = emas.pop();
                        const prevEma21 = emal.pop();
                        indicadorValor = `${ema9.toFixed(4)} / ${ema21.toFixed(4)}`;

                        if (prevEma9 < prevEma21 && ema9 > ema21) {
                            sinal = 'COMPRA (CRUZOU)'; cor = 'buy'; forca = 'FORTE';
                        } else if (prevEma9 > prevEma21 && ema9 < ema21) {
                            sinal = 'VENDA (CRUZOU)'; cor = 'sell'; forca = 'FORTE';
                        } else if (ema9 > ema21) {
                            sinal = 'Tendência de ALTA'; cor = 'neutro'; forca = 'Média';
                        } else if (ema9 < ema21) {
                            sinal = 'Tendência de BAIXA'; cor = 'neutro'; forca = 'Média';
                        }
                        break;
                    }

                    // ESTRATÉGIA 4: SuperTrend Pro
                    case '4': {
                        indicadorNome = 'SuperTrend(10,3)';
                        const stInput = { high, low, close, period: 10, multiplier: 3 };
                        const stResult = ti.Supertrend.calculate(stInput);
                        const st = stResult.pop();
                        const prevSt = stResult.pop();
                        indicadorValor = st.direction === 1 ? 'ALTA' : 'BAIXA';

                        if (st.direction === 1 && prevSt.direction === -1) {
                            sinal = 'COMPRA (FLIP)'; cor = 'buy'; forca = 'FORTE';
                        } else if (st.direction === -1 && prevSt.direction === 1) {
                            sinal = 'VENDA (FLIP)'; cor = 'sell'; forca = 'FORTE';
                        } else if (st.direction === 1) {
                            sinal = 'COMPRA'; cor = 'buy'; forca = 'Média';
                        } else if (st.direction === -1) {
                            sinal = 'VENDA'; cor = 'sell'; forca = 'Média';
                        }
                        break;
                    }
                    
                    // ESTRATÉGIA 5: Ichimoku Cloud
                    case '5': {
                        indicadorNome = 'Ichimoku';
                        const ichimoku = ti.IchimokuCloud.calculate({ high, low, conversionPeriod: 9, basePeriod: 26, spanPeriod: 52, displacement: 26 }).pop();
                        indicadorValor = ichimoku.spanA > ichimoku.spanB ? 'Nuvem Verde' : 'Nuvem Vermelha';
                        
                        const priceAboveCloud = preco > ichimoku.spanA && preco > ichimoku.spanB;
                        const priceBelowCloud = preco < ichimoku.spanA && preco < ichimoku.spanB;
                        const tenkanAboveKijun = ichimoku.conversion > ichimoku.base;

                        if (priceAboveCloud && tenkanAboveKijun) {
                            sinal = 'COMPRA FORTE'; cor = 'buy'; forca = 'FORTE';
                        } else if (priceBelowCloud && !tenkanAboveKijun) {
                            sinal = 'VENDA FORTE'; cor = 'sell'; forca = 'FORTE';
                        } else if (priceAboveCloud) {
                            sinal = 'COMPRA'; cor = 'buy'; forca = 'Média';
                        } else if (priceBelowCloud) {
                            sinal = 'VENDA'; cor = 'sell'; forca = 'Média';
                        }
                        break;
                    }
                }
            } catch (e) {
                console.error("Erro na análise técnica:", e);
                sinal = 'ERRO NA ANÁLISE';
                cor = 'erro';
                indicadorValor = e.message.substring(0, 20);
            }

            // Atualiza a UI
            const resultadoEl = document.getElementById('resultado');
            
            if (sinal !== 'ERRO NA ANÁLISE') {
                if (resultadoEl.innerText !== sinal) {
                    if (forca === 'FORTE') playForteSound();
                    if (cor === 'buy') playBuySound();
                    if (cor === 'sell') playSellSound();
                }
                resultadoEl.innerHTML = sinal;
                resultadoEl.className = cor;
            } else {
                 resultadoEl.innerHTML = sinal;
                 resultadoEl.className = cor;
            }
            
            document.getElementById('indicador-nome').innerText = indicadorNome;
            document.getElementById('indicador-valor').innerText = indicadorValor;
            document.getElementById('forca').innerText = forca;


            // Atualiza o link do WhatsApp
            const modo = (FORCE_SIMULATION_MODE || resultadoEl.className.includes('erro')) ? " (SIMULAÇÃO)" : "";
            ultimoSinal = `*SINAL DO ROBÔ FOREX BRASIL* 🇧🇷\n\n*AÇÃO:* ${sinal}${modo}\n*PAR:* ${par}\n*PREÇO:* ${preco.toFixed(5)}\n*TIMEFRAME:* ${document.getElementById('timeframe').value} min\n*ESTRATÉGIA:* ${document.getElementById('estrategia').options[document.getElementById('estrategia').selectedIndex].text}\n*HORÁRIO:* ${new Date().toLocaleString('pt-BR')}`;
            document.getElementById('whatsappBtn').href = `https://wa.me/?text=${encodeURIComponent(ultimoSinal)}`;
        }

        // Função de inicialização da página
        function inicializarPagina() {
            if (criarGrafico()) { 
                if (FORCE_SIMULATION_MODE) {
                    carregarGraficoSimulado(false); 
                } else {
                    buscarDados(false);    
                    conectarWebSocket(); 
                }
            }
        }
        
        window.onload = inicializarPagina;
        
    </script>
</body>
</html>

