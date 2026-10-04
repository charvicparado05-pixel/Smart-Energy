<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ENERGY METER DASHBOARD BY:CHRVC</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Chart.js CDN -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;600;800&family=Rajdhani:wght@500;600;700&display=swap');
        
        body {
            font-family: 'Rajdhani', sans-serif;
            background-color: #0b0f19;
            color: #e2e8f0;
        }
        
        .font-orbitron {
            font-family: 'Orbitron', sans-serif;
        }

        .glow-green {
            box-shadow: 0 0 15px rgba(16, 185, 129, 0.3);
        }
        
        .glow-red {
            box-shadow: 0 0 20px rgba(239, 68, 68, 0.5);
        }

        .glass-card {
            background: rgba(17, 24, 39, 0.85);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.08);
        }

        /* Pulse Animations */
        @keyframes pulse-fast {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.3; }
        }
        .animate-pulse-fast {
            animation: pulse-fast 0.8s infinite;
        }
    </style>
</head>
<body class="min-h-screen flex flex-col justify-between">

    <!-- TOP NAVIGATION HEADER -->
    <header class="border-b border-gray-800 bg-gray-900/90 sticky top-0 z-50 backdrop-blur">
        <div class="max-w-7xl mx-auto px-4 py-3 flex flex-wrap justify-between items-center gap-4">
            <div class="flex items-center space-x-3">
                <div class="bg-amber-500/20 p-2 rounded-lg border border-amber-500/40">
                    <i class="fa-solid fa-bolt text-amber-400 text-2xl"></i>
                </div>
                <div>
                    <h1 class="text-xl md:text-2xl font-extrabold font-orbitron text-transparent bg-clip-text bg-gradient-to-r from-amber-400 via-orange-400 to-yellow-200">
                        ENERGY METER DASHBOARD BY:CHRVC
                    </h1>
                    <p class="text-xs text-gray-400">Real-Time Load Monitoring & Automated Fault Disconnection System</p>
                </div>
            </div>

            <!-- WS CONNECT HUB -->
            <div class="flex items-center space-x-2 bg-gray-950 p-1.5 rounded-lg border border-gray-800 text-sm w-full md:w-auto">
                <span class="text-xs text-gray-400 pl-2">ESP32 IP:</span>
                <input type="text" id="wsIpInput" value="192.168.1.15" class="bg-gray-800 text-amber-400 px-2 py-1 rounded w-32 focus:outline-none focus:ring-1 focus:ring-amber-500 font-mono text-xs">
                <button id="wsConnectBtn" onclick="toggleWebSocketConnection()" class="bg-amber-500 hover:bg-amber-600 text-black font-semibold px-3 py-1 rounded text-xs transition">
                    Connect
                </button>
                <div class="h-4 w-px bg-gray-800 mx-1"></div>
                <button id="simToggleBtn" onclick="toggleSimulationMode()" class="bg-gray-800 hover:bg-gray-700 text-gray-300 border border-gray-700 px-2.5 py-1 rounded text-xs transition">
                    Sim Mode: <span id="simStatusText" class="text-emerald-400 font-bold">ON</span>
                </button>
            </div>
        </div>
    </header>

    <!-- MAIN CONTENT BODY -->
    <main class="max-w-7xl mx-auto px-4 py-6 w-full space-y-6">

        <!-- STATUS BAR & SYSTEM ALERTS -->
        <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
            <!-- RELAY HARDWARE STATUS -->
            <div id="relayCard" class="glass-card rounded-xl p-4 flex items-center justify-between border-l-4 border-emerald-500 transition-all duration-300">
                <div class="flex items-center space-x-3">
                    <div id="relayIconBg" class="w-12 h-12 rounded-full bg-emerald-500/20 flex items-center justify-center border border-emerald-500/40">
                        <i id="relayIcon" class="fa-solid fa-power-off text-emerald-400 text-xl"></i>
                    </div>
                    <div>
                        <div class="text-xs text-gray-400 font-bold uppercase tracking-wider">LOAD RELAY STATE</div>
                        <div id="relayStateText" class="text-xl font-bold font-orbitron text-emerald-400">CONNECTED (ON)</div>
                    </div>
                </div>
                <button id="manualRelayBtn" onclick="toggleManualRelay()" class="bg-emerald-500/20 hover:bg-emerald-500/30 text-emerald-400 border border-emerald-500/40 font-semibold px-3 py-1.5 rounded-lg text-xs transition">
                    DISCONNECT
                </button>
            </div>

            <!-- PROTECTION STATUS -->
            <div id="faultCard" class="glass-card rounded-xl p-4 flex items-center justify-between border-l-4 border-blue-500 transition-all duration-300">
                <div class="flex items-center space-x-3">
                    <div id="faultIconBg" class="w-12 h-12 rounded-full bg-blue-500/20 flex items-center justify-center border border-blue-500/40">
                        <i id="faultIcon" class="fa-solid fa-shield-halved text-blue-400 text-xl"></i>
                    </div>
                    <div>
                        <div class="text-xs text-gray-400 font-bold uppercase tracking-wider">AUTOMATED PROTECTION</div>
                        <div id="faultStatusText" class="text-lg font-bold font-orbitron text-blue-400">NORMAL OPERATIONAL</div>
                    </div>
                </div>
                <button onclick="resetFaultAlarm()" class="bg-gray-800 hover:bg-gray-700 text-gray-300 border border-gray-700 px-3 py-1.5 rounded-lg text-xs transition">
                    RESET
                </button>
            </div>

            <!-- WS CONNECTION STATUS -->
            <div class="glass-card rounded-xl p-4 flex items-center justify-between border-l-4 border-purple-500">
                <div class="flex items-center space-x-3">
                    <div class="w-12 h-12 rounded-full bg-purple-500/20 flex items-center justify-center border border-purple-500/40">
                        <i class="fa-solid fa-wifi text-purple-400 text-xl"></i>
                    </div>
                    <div>
                        <div class="text-xs text-gray-400 font-bold uppercase tracking-wider">ESP32 HARDWARE LINK</div>
                        <div id="wsLinkStatus" class="text-sm font-bold font-orbitron text-emerald-400">SIMULATION MODE ACTIVE</div>
                    </div>
                </div>
                <span id="pingBadge" class="text-xs font-mono text-gray-400 bg-gray-900 px-2 py-1 rounded border border-gray-800">0 ms</span>
            </div>
        </div>

        <!-- TELEMETRY METRIC CARDS GRID -->
        <div class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-6 gap-3">
            <!-- VOLTAGE -->
            <div class="glass-card p-4 rounded-xl border border-gray-800 hover:border-amber-500/50 transition">
                <div class="flex justify-between items-center text-gray-400 mb-1">
                    <span class="text-xs font-semibold uppercase">Voltage</span>
                    <i class="fa-solid fa-bolt-lightning text-amber-400 text-sm"></i>
                </div>
                <div class="text-2xl font-bold font-orbitron text-white"><span id="valVoltage">220.4</span> <span class="text-xs font-normal text-gray-400">V</span></div>
                <div class="text-[10px] text-gray-500 mt-1">Grid Rating: 220V AC</div>
            </div>

            <!-- CURRENT -->
            <div class="glass-card p-4 rounded-xl border border-gray-800 hover:border-amber-500/50 transition">
                <div class="flex justify-between items-center text-gray-400 mb-1">
                    <span class="text-xs font-semibold uppercase">Current</span>
                    <i class="fa-solid fa-wave-square text-cyan-400 text-sm"></i>
                </div>
                <div class="text-2xl font-bold font-orbitron text-white"><span id="valCurrent">4.25</span> <span class="text-xs font-normal text-gray-400">A</span></div>
                <div class="text-[10px] text-gray-500 mt-1">Max Threshold: <span id="lblMaxCurrent">15.0</span>A</div>
            </div>

            <!-- ACTIVE POWER -->
            <div class="glass-card p-4 rounded-xl border border-gray-800 hover:border-amber-500/50 transition">
                <div class="flex justify-between items-center text-gray-400 mb-1">
                    <span class="text-xs font-semibold uppercase">Active Power</span>
                    <i class="fa-solid fa-plug text-emerald-400 text-sm"></i>
                </div>
                <div class="text-2xl font-bold font-orbitron text-white"><span id="valPower">889.8</span> <span class="text-xs font-normal text-gray-400">W</span></div>
                <div class="text-[10px] text-gray-500 mt-1">Real-time Load</div>
            </div>

            <!-- POWER FACTOR -->
            <div class="glass-card p-4 rounded-xl border border-gray-800 hover:border-amber-500/50 transition">
                <div class="flex justify-between items-center text-gray-400 mb-1">
                    <span class="text-xs font-semibold uppercase">Power Factor</span>
                    <i class="fa-solid fa-chart-line text-purple-400 text-sm"></i>
                </div>
                <div class="text-2xl font-bold font-orbitron text-white"><span id="valPF">0.95</span></div>
                <div class="text-[10px] text-gray-500 mt-1">Target: > 0.85</div>
            </div>

            <!-- ENERGY ACCUMULATED -->
            <div class="glass-card p-4 rounded-xl border border-gray-800 hover:border-amber-500/50 transition">
                <div class="flex justify-between items-center text-gray-400 mb-1">
                    <span class="text-xs font-semibold uppercase">Total Energy</span>
                    <i class="fa-solid fa-gauge-high text-yellow-400 text-sm"></i>
                </div>
                <div class="text-2xl font-bold font-orbitron text-white"><span id="valEnergy">1.482</span> <span class="text-xs font-normal text-gray-400">kWh</span></div>
                <div class="text-[10px] text-gray-500 mt-1">Non-Volatile Counter</div>
            </div>

            <!-- ESTIMATED COST -->
            <div class="glass-card p-4 rounded-xl border border-gray-800 hover:border-amber-500/50 transition">
                <div class="flex justify-between items-center text-gray-400 mb-1">
                    <span class="text-xs font-semibold uppercase">Est. Cost</span>
                    <i class="fa-solid fa-peseta-sign text-emerald-400 text-sm"></i>
                </div>
                <div class="text-2xl font-bold font-orbitron text-emerald-400">₱<span id="valCost">17.04</span></div>
                <div class="text-[10px] text-gray-500 mt-1">Rate: ₱11.50 / kWh</div>
            </div>
        </div>

        <!-- CHARTS & FAULT INJECTION BENCH -->
        <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
            
            <!-- LIVE OSCILLATING CHART -->
            <div class="lg:col-span-2 glass-card p-5 rounded-2xl border border-gray-800 flex flex-col justify-between">
                <div class="flex justify-between items-center mb-4">
                    <div>
                        <h2 class="text-lg font-bold font-orbitron text-gray-200">Real-Time Load Waveform & Telemetry Stream</h2>
                        <p class="text-xs text-gray-400">Live monitoring of Voltage (V), Current (A), and Active Power (W)</p>
                    </div>
                    <div class="flex items-center space-x-2 text-xs">
                        <span class="inline-block w-3 h-3 bg-amber-400 rounded-full"></span><span class="text-gray-400 mr-2">Voltage</span>
                        <span class="inline-block w-3 h-3 bg-cyan-400 rounded-full"></span><span class="text-gray-400 mr-2">Current</span>
                        <span class="inline-block w-3 h-3 bg-emerald-400 rounded-full"></span><span class="text-gray-400">Power</span>
                    </div>
                </div>
                <div class="relative w-full h-72">
                    <canvas id="realtimeChart"></canvas>
                </div>
            </div>

            <!-- FAULT INJECTOR & HARDWARE CONTROLS -->
            <div class="glass-card p-5 rounded-2xl border border-gray-800 space-y-5">
                <div>
                    <h2 class="text-lg font-bold font-orbitron text-gray-200 flex items-center justify-between">
                        <span>Hardware Fault Injector</span>
                        <span class="text-xs font-normal bg-red-500/20 text-red-400 px-2 py-0.5 rounded border border-red-500/30">Test Bench</span>
                    </h2>
                    <p class="text-xs text-gray-400">Simulate over-limit conditions to evaluate automated relay trip safety.</p>
                </div>

                <!-- FAULT TEST BUTTONS -->
                <div class="grid grid-cols-2 gap-2">
                    <button onclick="injectFault('OVERCURRENT')" class="bg-red-500/10 hover:bg-red-500/20 text-red-400 border border-red-500/30 p-2.5 rounded-lg text-xs font-semibold flex flex-col items-center gap-1 transition">
                        <i class="fa-solid fa-triangle-exclamation text-base"></i>
                        Overcurrent (28.5A)
                    </button>
                    <button onclick="injectFault('OVERVOLTAGE')" class="bg-orange-500/10 hover:bg-orange-500/20 text-orange-400 border border-orange-500/30 p-2.5 rounded-lg text-xs font-semibold flex flex-col items-center gap-1 transition">
                        <i class="fa-solid fa-bolt text-base"></i>
                        Overvoltage (275V)
                    </button>
                    <button onclick="injectFault('SHORT_CIRCUIT')" class="bg-yellow-500/10 hover:bg-yellow-500/20 text-yellow-400 border border-yellow-500/30 p-2.5 rounded-lg text-xs font-semibold flex flex-col items-center gap-1 transition">
                        <i class="fa-solid fa-burst text-base"></i>
                        Short Circuit (100A)
                    </button>
                    <button onclick="injectFault('UNDERVOLTAGE')" class="bg-blue-500/10 hover:bg-blue-500/20 text-blue-400 border border-blue-500/30 p-2.5 rounded-lg text-xs font-semibold flex flex-col items-center gap-1 transition">
                        <i class="fa-solid fa-arrow-trend-down text-base"></i>
                        Undervoltage (160V)
                    </button>
                </div>

                <!-- SAFETY THRESHOLDS CALIBRATION -->
                <div class="space-y-3 pt-3 border-t border-gray-800">
                    <h3 class="text-xs font-bold text-gray-300 uppercase tracking-wider">Safety Limit Calibration</h3>
                    
                    <div>
                        <div class="flex justify-between text-xs text-gray-400 mb-1">
                            <span>Max Overcurrent Threshold:</span>
                            <span id="sliderCurrentVal" class="text-amber-400 font-bold">15.0 A</span>
                        </div>
                        <input type="range" id="sliderCurrent" min="1" max="30" value="15" step="0.5" oninput="updateThresholds()" class="w-full accent-amber-500 bg-gray-800 rounded">
                    </div>

                    <div>
                        <div class="flex justify-between text-xs text-gray-400 mb-1">
                            <span>Max Overvoltage Threshold:</span>
                            <span id="sliderVoltageVal" class="text-amber-400 font-bold">250 V</span>
                        </div>
                        <input type="range" id="sliderVoltage" min="200" max="280" value="250" step="1" oninput="updateThresholds()" class="w-full accent-amber-500 bg-gray-800 rounded">
                    </div>
                </div>
            </div>
        </div>

        <!-- FAULT LOGS TABLE SECTION -->
        <div class="glass-card p-5 rounded-2xl border border-gray-800">
            <div class="flex flex-wrap justify-between items-center gap-4 mb-4">
                <div>
                    <h2 class="text-lg font-bold font-orbitron text-gray-200">Automated Fault Disconnection Logs</h2>
                    <p class="text-xs text-gray-400">History of real-time electrical anomalies and automated protective actions</p>
                </div>
                <div class="flex space-x-2">
                    <button onclick="exportCSVLogs()" class="bg-gray-800 hover:bg-gray-700 text-gray-300 border border-gray-700 px-3 py-1.5 rounded-lg text-xs font-semibold transition flex items-center gap-2">
                        <i class="fa-solid fa-file-csv text-emerald-400"></i> Export CSV
                    </button>
                    <button onclick="clearFaultLogs()" class="bg-gray-800 hover:bg-gray-700 text-red-400 border border-gray-700 px-3 py-1.5 rounded-lg text-xs font-semibold transition">
                        Clear Logs
                    </button>
                </div>
            </div>

            <div class="overflow-x-auto">
                <table class="w-full text-left text-xs text-gray-300">
                    <thead class="bg-gray-950 text-gray-400 uppercase font-mono">
                        <tr>
                            <th class="p-3 border-b border-gray-800">Timestamp</th>
                            <th class="p-3 border-b border-gray-800">Event / Fault Type</th>
                            <th class="p-3 border-b border-gray-800">Telemetry Value</th>
                            <th class="p-3 border-b border-gray-800">Threshold Limit</th>
                            <th class="p-3 border-b border-gray-800">Relay Action</th>
                            <th class="p-3 border-b border-gray-800">Status</th>
                        </tr>
                    </thead>
                    <tbody id="faultLogTableBody" class="divide-y divide-gray-800 font-mono">
                        <!-- Dynamic Fault Logs populate here -->
                    </tbody>
                </table>
            </div>
        </div>

    </main>

    <!-- FOOTER -->
    <footer class="border-t border-gray-800 py-4 text-center text-xs text-gray-500 bg-gray-950">
        <p>ENERGY METER DASHBOARD BY:CHRVC | Designed for ESP32 & PZEM-004T Integration</p>
    </footer>

    <!-- JAVASCRIPT SYSTEM ENGINE -->
    <script>
        // Global State
        let isSimulation = true;
        let relayActive = true;
        let isFaulted = false;
        let currentFaultType = "NORMAL";
        let ws = null;

        // Telemetry Data Points
        let telemetry = {
            voltage: 220.0,
            current: 4.25,
            power: 889.8,
            pf: 0.95,
            energy: 1.482,
            cost: 17.04,
            frequency: 60.0
        };

        // Safety Threshold Limits
        let thresholds = {
            maxCurrent: 15.0,
            maxVoltage: 250.0,
            minVoltage: 180.0
        };

        // Log Storage
        let faultLogs = [];

        // Chart.js Real-time Setup
        let chartContext = document.getElementById('realtimeChart').getContext('2d');
        let chartLabels = Array(20).fill('');
        let chartVoltageData = Array(20).fill(220);
        let chartCurrentData = Array(20).fill(4.25);
        let chartPowerData = Array(20).fill(889);

        let realtimeChart = new Chart(chartContext, {
            type: 'line',
            data: {
                labels: chartLabels,
                datasets: [
                    {
                        label: 'Voltage (V)',
                        data: chartVoltageData,
                        borderColor: '#fbbf24',
                        borderWidth: 2,
                        fill: false,
                        tension: 0.3,
                        pointRadius: 0
                    },
                    {
                        label: 'Current (A)',
                        data: chartCurrentData,
                        borderColor: '#22d3ee',
                        borderWidth: 2,
                        fill: false,
                        tension: 0.3,
                        pointRadius: 0
                    },
                    {
                        label: 'Power (W)',
                        data: chartPowerData,
                        borderColor: '#10b981',
                        borderWidth: 1.5,
                        borderDash: [4, 4],
                        fill: false,
                        tension: 0.3,
                        pointRadius: 0
                    }
                ]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                animation: false,
                scales: {
                    x: { display: false },
                    y: {
                        grid: { color: 'rgba(255, 255, 255, 0.05)' },
                        ticks: { color: '#9ca3af', font: { size: 10 } }
                    }
                },
                plugins: {
                    legend: { display: false }
                }
            }
        });

        // Initialize App
        window.onload = function() {
            startSimulationEngine();
            addLogEntry('SYSTEM START', 'Dashboard Initialized', 'N/A', 'N/A', 'Relay Armed', 'NORMAL');
        };

        // Simulation Data Generator Loop
        function startSimulationEngine() {
            setInterval(() => {
                if (!isSimulation) return;

                if (relayActive && !isFaulted) {
                    // Small real-time fluctuations
                    telemetry.voltage = +(218 + Math.random() * 5).toFixed(1);
                    telemetry.current = +(3.8 + Math.random() * 0.8).toFixed(2);
                    telemetry.power = +(telemetry.voltage * telemetry.current * telemetry.pf).toFixed(1);
                    telemetry.energy = +(telemetry.energy + 0.0001).toFixed(4);
                    telemetry.cost = +(telemetry.energy * 11.50).toFixed(2);
                } else if (!relayActive) {
                    // Relay tripped or off -> zero current/power
                    telemetry.voltage = 220.0;
                    telemetry.current = 0.00;
                    telemetry.power = 0.0;
                }

                checkAutomatedProtectionLogic();
                updateUI();
                pushChartData();

            }, 1000);
        }

        // Check Automated Disconnection Logic
        function checkAutomatedProtectionLogic() {
            if (!relayActive) return;

            if (telemetry.current > thresholds.maxCurrent) {
                triggerAutomatedTrip('OVERCURRENT TRIP', `${telemetry.current} A`, `${thresholds.maxCurrent} A`);
            } else if (telemetry.voltage > thresholds.maxVoltage) {
                triggerAutomatedTrip('OVERVOLTAGE TRIP', `${telemetry.voltage} V`, `${thresholds.maxVoltage} V`);
            } else if (telemetry.voltage < thresholds.minVoltage) {
                triggerAutomatedTrip('UNDERVOLTAGE TRIP', `${telemetry.voltage} V`, `${thresholds.minVoltage} V`);
            }
        }

        // Trigger Automated Trip Action
        function triggerAutomatedTrip(faultType, valStr, limitStr) {
            relayActive = false;
            isFaulted = true;
            currentFaultType = faultType;

            // Send command to physical ESP32 if connected
            if (ws && ws.readyState === WebSocket.OPEN) {
                ws.send("RELAY_OFF");
            }

            addLogEntry(
                new Date().toLocaleTimeString(),
                faultType,
                valStr,
                limitStr,
                'DISCONNECTED (TRIPPED)',
                'TRIPPED'
            );

            updateUI();
        }

        // Push new data points to chart
        function pushChartData() {
            chartVoltageData.shift();
            chartCurrentData.shift();
            chartPowerData.shift();

            chartVoltageData.push(telemetry.voltage);
            chartCurrentData.push(telemetry.current * 10); // scale current for visual alignment
            chartPowerData.push(telemetry.power / 10);     // scale power for visual alignment

            realtimeChart.update();
        }

        // Update Dashboard Elements
        function updateUI() {
            document.getElementById('valVoltage').innerText = telemetry.voltage;
            document.getElementById('valCurrent').innerText = telemetry.current;
            document.getElementById('valPower').innerText = telemetry.power;
            document.getElementById('valPF').innerText = telemetry.pf;
            document.getElementById('valEnergy').innerText = telemetry.energy;
            document.getElementById('valCost').innerText = telemetry.cost;

            // Update Relay Display
            const relayCard = document.getElementById('relayCard');
            const relayStateText = document.getElementById('relayStateText');
            const relayIconBg = document.getElementById('relayIconBg');
            const relayIcon = document.getElementById('relayIcon');
            const manualRelayBtn = document.getElementById('manualRelayBtn');

            if (relayActive) {
                relayCard.className = "glass-card rounded-xl p-4 flex items-center justify-between border-l-4 border-emerald-500 transition-all duration-300";
                relayStateText.innerText = "CONNECTED (ON)";
                relayStateText.className = "text-xl font-bold font-orbitron text-emerald-400";
                relayIconBg.className = "w-12 h-12 rounded-full bg-emerald-500/20 flex items-center justify-center border border-emerald-500/40";
                relayIcon.className = "fa-solid fa-power-off text-emerald-400 text-xl";
                manualRelayBtn.innerText = "DISCONNECT";
                manualRelayBtn.className = "bg-emerald-500/20 hover:bg-emerald-500/30 text-emerald-400 border border-emerald-500/40 font-semibold px-3 py-1.5 rounded-lg text-xs transition";
            } else {
                relayCard.className = "glass-card rounded-xl p-4 flex items-center justify-between border-l-4 border-red-500 glow-red transition-all duration-300";
                relayStateText.innerText = "DISCONNECTED (OFF)";
                relayStateText.className = "text-xl font-bold font-orbitron text-red-500 animate-pulse-fast";
                relayIconBg.className = "w-12 h-12 rounded-full bg-red-500/20 flex items-center justify-center border border-red-500/40";
                relayIcon.className = "fa-solid fa-triangle-exclamation text-red-500 text-xl";
                manualRelayBtn.innerText = "RE-CONNECT";
                manualRelayBtn.className = "bg-red-500/20 hover:bg-red-500/30 text-red-400 border border-red-500/40 font-semibold px-3 py-1.5 rounded-lg text-xs transition";
            }

            // Update Fault Status Display
            const faultCard = document.getElementById('faultCard');
            const faultStatusText = document.getElementById('faultStatusText');
            const faultIconBg = document.getElementById('faultIconBg');
            const faultIcon = document.getElementById('faultIcon');

            if (!isFaulted) {
                faultCard.className = "glass-card rounded-xl p-4 flex items-center justify-between border-l-4 border-blue-500";
                faultStatusText.innerText = "NORMAL OPERATIONAL";
                faultStatusText.className = "text-lg font-bold font-orbitron text-blue-400";
                faultIconBg.className = "w-12 h-12 rounded-full bg-blue-500/20 flex items-center justify-center border border-blue-500/40";
                faultIcon.className = "fa-solid fa-shield-halved text-blue-400 text-xl";
            } else {
                faultCard.className = "glass-card rounded-xl p-4 flex items-center justify-between border-l-4 border-red-500 glow-red";
                faultStatusText.innerText = currentFaultType;
                faultStatusText.className = "text-lg font-bold font-orbitron text-red-400 animate-pulse-fast";
                faultIconBg.className = "w-12 h-12 rounded-full bg-red-500/20 flex items-center justify-center border border-red-500/40";
                faultIcon.className = "fa-solid fa-bell text-red-400 text-xl animate-bounce";
            }
        }

        // Toggle Manual Relay State
        function toggleManualRelay() {
            relayActive = !relayActive;
            
            if (relayActive) {
                isFaulted = false;
                currentFaultType = "NORMAL";
            }

            // Send command over WebSocket to ESP32
            if (ws && ws.readyState === WebSocket.OPEN) {
                ws.send(relayActive ? "RELAY_ON" : "RELAY_OFF");
            }

            addLogEntry(
                new Date().toLocaleTimeString(),
                relayActive ? 'MANUAL RE-CONNECT' : 'MANUAL DISCONNECT',
                `${telemetry.current} A`,
                'User Command',
                relayActive ? 'CONNECTED' : 'DISCONNECTED',
                relayActive ? 'NORMAL' : 'MANUAL_OFF'
            );

            updateUI();
        }

        // Reset Alarm Condition
        function resetFaultAlarm() {
            isFaulted = false;
            currentFaultType = "NORMAL";
            relayActive = true;

            if (ws && ws.readyState === WebSocket.OPEN) {
                ws.send("RELAY_ON");
            }

            addLogEntry(
                new Date().toLocaleTimeString(),
                'SYSTEM RESET',
                'N/A',
                'User Reset',
                'RELAY RE-ARMED',
                'NORMAL'
            );

            updateUI();
        }

        // Inject Fault Condition for Demonstration
        function injectFault(type) {
            if (type === 'OVERCURRENT') {
                telemetry.current = 28.5;
                telemetry.power = +(telemetry.voltage * telemetry.current * telemetry.pf).toFixed(1);
            } else if (type === 'OVERVOLTAGE') {
                telemetry.voltage = 275.0;
            } else if (type === 'SHORT_CIRCUIT') {
                telemetry.current = 100.0;
                telemetry.voltage = 190.0;
            } else if (type === 'UNDERVOLTAGE') {
                telemetry.voltage = 160.0;
            }

            checkAutomatedProtectionLogic();
            updateUI();
        }

        // Update Calibration Limits from Sliders
        function updateThresholds() {
            thresholds.maxCurrent = parseFloat(document.getElementById('sliderCurrent').value);
            thresholds.maxVoltage = parseFloat(document.getElementById('sliderVoltage').value);

            document.getElementById('sliderCurrentVal').innerText = thresholds.maxCurrent.toFixed(1) + " A";
            document.getElementById('sliderVoltageVal').innerText = thresholds.maxVoltage + " V";
            document.getElementById('lblMaxCurrent').innerText = thresholds.maxCurrent.toFixed(1);
        }

        // Add Log Entry to Data Table
        function addLogEntry(time, event, value, limit, action, status) {
            const entry = { time, event, value, limit, action, status };
            faultLogs.unshift(entry);

            const tbody = document.getElementById('faultLogTableBody');
            const row = document.createElement('tr');
            row.className = status === 'TRIPPED' ? 'bg-red-500/10 text-red-300' : 'hover:bg-gray-900/50';

            row.innerHTML = `
                <td class="p-3 border-b border-gray-800">${time}</td>
                <td class="p-3 border-b border-gray-800 font-bold">${event}</td>
                <td class="p-3 border-b border-gray-800">${value}</td>
                <td class="p-3 border-b border-gray-800">${limit}</td>
                <td class="p-3 border-b border-gray-800">${action}</td>
                <td class="p-3 border-b border-gray-800">
                    <span class="px-2 py-0.5 rounded text-[10px] font-bold ${status === 'TRIPPED' ? 'bg-red-500/20 text-red-400 border border-red-500/40' : 'bg-emerald-500/20 text-emerald-400 border border-emerald-500/40'}">
                        ${status}
                    </span>
                </td>
            `;

            tbody.insertBefore(row, tbody.firstChild);
        }

        // Clear Table Logs
        function clearFaultLogs() {
            faultLogs = [];
            document.getElementById('faultLogTableBody').innerHTML = '';
        }

        // Export Logs as CSV File
        function exportCSVLogs() {
            if (faultLogs.length === 0) {
                alert("No log records available to export!");
                return;
            }

            let csvContent = "data:text/csv;charset=utf-8,Timestamp,Event,Telemetry Value,Threshold Limit,Action,Status\n";
            faultLogs.forEach(log => {
                csvContent += `"${log.time}","${log.event}","${log.value}","${log.limit}","${log.action}","${log.status}"\n`;
            });

            const encodedUri = encodeURI(csvContent);
            const link = document.createElement("a");
            link.setAttribute("href", encodedUri);
            link.setAttribute("download", `Energy_Meter_Fault_Logs_${Date.now()}.csv`);
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
        }

        // Toggle Simulation Mode
        function toggleSimulationMode() {
            isSimulation = !isSimulation;
            const btnText = document.getElementById('simStatusText');
            const wsStatus = document.getElementById('wsLinkStatus');

            if (isSimulation) {
                btnText.innerText = "ON";
                btnText.className = "text-emerald-400 font-bold";
                wsStatus.innerText = "SIMULATION MODE ACTIVE";
                wsStatus.className = "text-sm font-bold font-orbitron text-emerald-400";
            } else {
                btnText.innerText = "OFF";
                btnText.className = "text-gray-400 font-bold";
                wsStatus.innerText = "PAUSED (AWAITING ESP32)";
                wsStatus.className = "text-sm font-bold font-orbitron text-amber-400";
            }
        }

        // Toggle Real ESP32 WebSocket Connection
        function toggleWebSocketConnection() {
            const ip = document.getElementById('wsIpInput').value;
            const btn = document.getElementById('wsConnectBtn');
            const wsStatus = document.getElementById('wsLinkStatus');

            if (ws && (ws.readyState === WebSocket.OPEN || ws.readyState === WebSocket.CONNECTING)) {
                ws.close();
                btn.innerText = "Connect";
                btn.className = "bg-amber-500 hover:bg-amber-600 text-black font-semibold px-3 py-1 rounded text-xs transition";
                return;
            }

            btn.innerText = "Connecting...";
            wsStatus.innerText = "CONNECTING...";
            wsStatus.className = "text-sm font-bold font-orbitron text-amber-400";

            try {
                ws = new WebSocket(`ws://${ip}:81/`);

                ws.onopen = function() {
                    btn.innerText = "Disconnect";
                    btn.className = "bg-red-500 hover:bg-red-600 text-white font-semibold px-3 py-1 rounded text-xs transition";
                    wsStatus.innerText = "CONNECTED TO ESP32";
                    wsStatus.className = "text-sm font-bold font-orbitron text-emerald-400";
                    isSimulation = false;
                    document.getElementById('simStatusText').innerText = "OFF";
                };

                ws.onmessage = function(event) {
                    const data = JSON.parse(event.data);
                    telemetry.voltage = data.voltage || 0;
                    telemetry.current = data.current || 0;
                    telemetry.power = data.power || 0;
                    telemetry.energy = data.energy || 0;
                    telemetry.pf = data.pf || 0.95;
                    relayActive = data.relay !== undefined ? data.relay : true;

                    updateUI();
                    pushChartData();
                };

                ws.onerror = function() {
                    alert("WebSocket Connection Error! Check ESP32 IP & Wi-Fi Network.");
                    wsStatus.innerText = "CONNECTION FAILED";
                    wsStatus.className = "text-sm font-bold font-orbitron text-red-500";
                    btn.innerText = "Connect";
                    btn.className = "bg-amber-500 hover:bg-amber-600 text-black font-semibold px-3 py-1 rounded text-xs transition";
                };

            } catch (err) {
                alert("Invalid WebSocket Configuration!");
            }
        }
    </script>
</body>
</html>
