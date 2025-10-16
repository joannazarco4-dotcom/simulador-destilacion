<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulador de Torre de Destilación</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        .btn-run {
            background-color: #2563eb; /* Azul estándar */
        }
        .btn-run:hover {
            background-color: #1d4ed8;
        }
    </style>
</head>
<body class="bg-slate-100 text-gray-800 flex flex-col items-center justify-center min-h-screen p-4">

    <div class="w-full max-w-6xl mx-auto">
        <h1 class="text-4xl font-bold text-center mb-2 text-slate-800">Simulador de Torre de Destilación</h1>
        <p class="text-center text-gray-600 mb-8">Una herramienta para visualizar la destilación binaria basada en el método McCabe-Thiele.</p>

        <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">

            <!-- Panel de Control -->
            <div class="lg:col-span-1 bg-white p-6 rounded-lg shadow-md">
                <h2 class="text-2xl font-semibold mb-6 border-b border-gray-200 pb-2">Parámetros de Entrada</h2>

                <div class="space-y-4">
                    <div>
                        <label for="mixtureSelect" class="block mb-2 text-sm font-medium text-gray-700">Seleccionar Mezcla Binaria</label>
                        <select id="mixtureSelect" class="w-full bg-gray-50 border border-gray-300 text-gray-900 rounded-lg p-2.5 focus:ring-blue-500 focus:border-blue-500 block">
                            <option value="custom">Personalizado</option>
                            <option value="benzene-toluene">Benceno - Tolueno</option>
                            <option value="methanol-water">Metanol - Agua</option>
                            <option value="ethanol-water">Etanol - Agua (Idealizado)</option>
                            <option value="hexane-heptane">n-Hexano - n-Heptano</option>
                        </select>
                    </div>
                    <div>
                        <label for="relativeVolatility" class="block mb-2 text-sm font-medium text-gray-700">Volatilidad Relativa (α)</label>
                        <input id="relativeVolatility" type="number" value="2.5" min="1.01" max="10" step="0.1" class="w-full bg-gray-50 border border-gray-300 text-gray-900 rounded-lg p-2.5 focus:ring-blue-500 focus:border-blue-500 block">
                    </div>
                    <div>
                        <label for="feedComposition" class="block mb-2 text-sm font-medium text-gray-700">Composición de Alimentación (zF)</label>
                        <input id="feedComposition" type="number" value="0.5" min="0.01" max="0.99" step="0.01" class="w-full bg-gray-50 border border-gray-300 text-gray-900 rounded-lg p-2.5 focus:ring-blue-500 focus:border-blue-500 block">
                    </div>
                    <div>
                        <label for="refluxRatio" class="block mb-2 text-sm font-medium text-gray-700">Relación de Reflujo (R)</label>
                        <input id="refluxRatio" type="number" value="1.5" min="0.1" max="10" step="0.1" class="w-full bg-gray-50 border border-gray-300 text-gray-900 rounded-lg p-2.5 focus:ring-blue-500 focus:border-blue-500 block">
                    </div>
                     <div>
                        <label for="qValue" class="block mb-2 text-sm font-medium text-gray-700">Calidad de la Alimentación (q)</label>
                        <input id="qValue" type="number" value="1.0" min="0" max="1.5" step="0.1" class="w-full bg-gray-50 border border-gray-300 text-gray-900 rounded-lg p-2.5 focus:ring-blue-500 focus:border-blue-500 block">
                        <p class="text-xs text-gray-500 mt-1">q=0: Vapor saturado, q=1: Líquido saturado</p>
                    </div>
                </div>

                <div class="mt-8">
                    <button id="runSimulation" class="w-full btn-run text-white font-bold py-3 px-4 rounded-lg transition duration-300 ease-in-out transform hover:scale-105">
                        Ejecutar Simulación
                    </button>
                </div>
            </div>

            <!-- Visualización y Resultados -->
            <div class="lg:col-span-2 space-y-6">
                <!-- Torre de Destilación y Gráfico -->
                <div class="bg-white p-4 rounded-lg shadow-md h-full">
                    <h2 class="text-xl font-semibold mb-4 text-center">Diagrama de Equilibrio y Etapas</h2>
                    <canvas id="mccabeThieleCanvas"></canvas>
                </div>

                <!-- Resultados -->
                <div class="bg-white p-6 rounded-lg shadow-md">
                    <h2 class="text-2xl font-semibold mb-4 border-b border-gray-200 pb-2">Resultados de la Simulación</h2>
                    <div id="resultsOutput" class="grid grid-cols-1 sm:grid-cols-3 gap-4 text-center">
                        <div>
                            <h3 class="text-lg font-medium text-blue-600">Etapas Teóricas</h3>
                            <p id="theoreticalStages" class="text-3xl font-bold">--</p>
                        </div>
                        <div>
                            <h3 class="text-lg font-medium text-green-600">Composición Destilado (x_D)</h3>
                            <p id="distillateComposition" class="text-3xl font-bold">--</p>
                        </div>
                        <div>
                            <h3 class="text-lg font-medium text-orange-600">Composición Fondos (x_B)</h3>
                            <p id="bottomsComposition" class="text-3xl font-bold">--</p>
                        </div>
                    </div>
                     <div id="message" class="text-center mt-4 text-red-600 font-medium"></div>
                </div>
            </div>

        </div>
    </div>

    <script>
        // --- Base de datos de mezclas (Valores de α promedio a 1 atm) ---
        const VLEData = {
            "benzene-toluene": { alpha: 2.4, name: "Benceno - Tolueno" },
            "methanol-water": { alpha: 3.5, name: "Metanol - Agua" }, // Valor promedio, es fuertemente no-ideal
            "ethanol-water": { alpha: 2.8, name: "Etanol - Agua (Idealizado)" }, // Muy no-ideal, forma azeótropo. Este es un valor simplificado.
            "hexane-heptane": { alpha: 2.2, name: "n-Hexano - n-Heptano" }
        };

        // --- Elementos del DOM ---
        const mixtureSelect = document.getElementById('mixtureSelect');
        const feedCompositionInput = document.getElementById('feedComposition');
        const refluxRatioInput = document.getElementById('refluxRatio');
        const relativeVolatilityInput = document.getElementById('relativeVolatility');
        const qValueInput = document.getElementById('qValue');
        const runButton = document.getElementById('runSimulation');
        const canvas = document.getElementById('mccabeThieleCanvas');
        const ctx = canvas.getContext('2d');
        
        // --- Salidas de resultados ---
        const theoreticalStagesEl = document.getElementById('theoreticalStages');
        const distillateCompositionEl = document.getElementById('distillateComposition');
        const bottomsCompositionEl = document.getElementById('bottomsComposition');
        const messageEl = document.getElementById('message');

        const desiredDistillateComp = 0.95;
        const desiredBottomsComp = 0.05;

        // --- Lógica de la interfaz ---
        mixtureSelect.addEventListener('change', (e) => {
            const selectedMixture = e.target.value;
            if (selectedMixture === 'custom') {
                relativeVolatilityInput.disabled = false;
                relativeVolatilityInput.value = 2.5; // Valor por defecto
            } else {
                relativeVolatilityInput.disabled = true;
                relativeVolatilityInput.value = VLEData[selectedMixture].alpha;
            }
        });
        
        // --- Función para dibujar el gráfico base ---
        function drawBaseChart() {
            const parentEl = canvas.parentElement;
            if (!parentEl) return;

            const width = parentEl.clientWidth * 0.95;
            const height = width * 0.8;
            canvas.width = width;
            canvas.height = height;

            const padding = 50;
            const chartWidth = width - 2 * padding;
            const chartHeight = height - 2 * padding;

            ctx.clearRect(0, 0, width, height);
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(0, 0, width, height);

            ctx.save();
            ctx.translate(padding, padding);

            // Ejes y etiquetas
            ctx.strokeStyle = '#6b7280';
            ctx.lineWidth = 1;
            ctx.beginPath();
            ctx.moveTo(0, 0);
            ctx.lineTo(0, chartHeight);
            ctx.lineTo(chartWidth, chartHeight);
            ctx.stroke();

            ctx.fillStyle = '#374151';
            ctx.font = '12px Inter';
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            ctx.fillText('Fracción molar en líquido (x)', chartWidth / 2, chartHeight + 30);
            ctx.save();
            ctx.translate(-35, chartHeight / 2);
            ctx.rotate(-Math.PI / 2);
            ctx.fillText('Fracción molar en vapor (y)', 0, 0);
            ctx.restore();

            // Marcas de los ejes
            for (let i = 0; i <= 10; i++) {
                const val = (i / 10).toFixed(1);
                const x = (i / 10) * chartWidth;
                ctx.fillText(val, x, chartHeight + 15);
                ctx.beginPath();
                ctx.moveTo(x, chartHeight);
                ctx.lineTo(x, chartHeight - 5);
                ctx.stroke();

                const y = chartHeight - (i / 10) * chartHeight;
                ctx.fillText(val, -20, y);
                ctx.beginPath();
                ctx.moveTo(0, y);
                ctx.lineTo(5, y);
                ctx.stroke();
            }

            // Línea y=x
            ctx.strokeStyle = '#d1d5db';
            ctx.setLineDash([5, 5]);
            ctx.beginPath();
            ctx.moveTo(0, chartHeight);
            ctx.lineTo(chartWidth, 0);
            ctx.stroke();
            ctx.setLineDash([]);
            
            ctx.restore();
        }

        // --- Función de simulación ---
        function runSimulation() {
            messageEl.textContent = '';
            drawBaseChart();

            const zF = parseFloat(feedCompositionInput.value);
            const R = parseFloat(refluxRatioInput.value);
            const alpha = parseFloat(relativeVolatilityInput.value);
            const q = parseFloat(qValueInput.value);
            const xD = desiredDistillateComp;
            const xB = desiredBottomsComp;

            const width = canvas.width;
            const height = canvas.height;
            const padding = 50;
            const chartWidth = width - 2 * padding;
            const chartHeight = height - 2 * padding;
            
            ctx.save();
            ctx.translate(padding, padding);

            const toCanvasX = (x) => x * chartWidth;
            const toCanvasY = (y) => chartHeight - y * chartHeight;

            // 1. Curva de equilibrio
            ctx.strokeStyle = '#2563eb'; // Azul
            ctx.lineWidth = 2;
            ctx.beginPath();
            for (let i = 0; i <= 100; i++) {
                const x = i / 100;
                const y = (alpha * x) / (1 + (alpha - 1) * x);
                if (i === 0) ctx.moveTo(toCanvasX(x), toCanvasY(y));
                else ctx.lineTo(toCanvasX(x), toCanvasY(y));
            }
            ctx.stroke();

            // 2. Línea q (alimentación)
            let qSlope = (q === 1) ? Infinity : q / (q - 1);
            let qIntercept = zF * (1 - qSlope);

            ctx.strokeStyle = '#f59e0b'; // Naranja
            ctx.beginPath();
            ctx.moveTo(toCanvasX(zF), toCanvasY(zF));
            const y_eq_at_zF = (alpha * zF) / (1 + (alpha - 1) * zF);
            let x_intersect_eq = (q === 1) ? zF : (y_eq_at_zF - qIntercept) / qSlope;
             if (q === 1) {
                ctx.lineTo(toCanvasX(zF), toCanvasY(y_eq_at_zF));
            } else {
                 ctx.lineTo(toCanvasX(x_intersect_eq), toCanvasY(y_eq_at_zF));
            }
            ctx.stroke();
            
            // 3. Línea de operación de enriquecimiento
            const rect_slope = R / (R + 1);
            const rect_intercept = xD / (R + 1);
            ctx.strokeStyle = '#16a34a'; // Verde
            ctx.beginPath();
            ctx.moveTo(toCanvasX(xD), toCanvasY(xD));
            
            let intersectX = (q === 1) ? zF : (rect_intercept - qIntercept) / (qSlope - rect_slope);
            let intersectY = rect_slope * intersectX + rect_intercept;
            ctx.lineTo(toCanvasX(intersectX), toCanvasY(intersectY));
            ctx.stroke();
            
            // 4. Línea de operación de agotamiento
            ctx.strokeStyle = '#dc2626'; // Rojo
            ctx.beginPath();
            ctx.moveTo(toCanvasX(xB), toCanvasY(xB));
            ctx.lineTo(toCanvasX(intersectX), toCanvasY(intersectY));
            ctx.stroke();

            // 5. Etapas
            let stages = 0;
            let currentX = xD, currentY = xD;
            ctx.strokeStyle = '#7c3aed'; // Violeta
            ctx.lineWidth = 1;

            if (rect_intercept <= 0 || R < 0) {
                 messageEl.textContent = 'Condición inválida o de reflujo mínimo. Aumente la Relación de Reflujo.';
                 theoreticalStagesEl.textContent = '∞';
                 distillateCompositionEl.textContent = desiredDistillateComp.toFixed(2);
                 bottomsCompositionEl.textContent = desiredBottomsComp.toFixed(2);
                 ctx.restore();
                 return;
            }

            while (currentX > xB && stages < 50) {
                const equilibriumX = findEquilibriumX(currentY, alpha);
                ctx.beginPath();
                ctx.moveTo(toCanvasX(currentX), toCanvasY(currentY));
                ctx.lineTo(toCanvasX(equilibriumX), toCanvasY(currentY));
                ctx.stroke();
                
                currentX = equilibriumX;
                
                let nextY;
                if (currentX > intersectX) {
                    nextY = rect_slope * currentX + rect_intercept;
                } else {
                    const strip_slope = (intersectY - xB) / (intersectX - xB);
                    const strip_intercept = xB - strip_slope * xB;
                    nextY = strip_slope * currentX + strip_intercept;
                }
                ctx.beginPath();
                ctx.moveTo(toCanvasX(currentX), toCanvasY(currentY));
                ctx.lineTo(toCanvasX(currentX), toCanvasY(nextY));
                ctx.stroke();
                currentY = nextY;
                stages++;
            }
            
            ctx.restore();

            theoreticalStagesEl.textContent = stages;
            distillateCompositionEl.textContent = xD.toFixed(2);
            bottomsCompositionEl.textContent = xB.toFixed(2);
        }
        
        function findEquilibriumX(y, alpha) {
            return y / (alpha - y * (alpha - 1));
        }

        // --- Eventos y Carga Inicial ---
        runButton.addEventListener('click', runSimulation);
        window.addEventListener('resize', drawBaseChart);

        window.onload = () => {
            mixtureSelect.value = "benzene-toluene";
            relativeVolatilityInput.value = VLEData["benzene-toluene"].alpha;
            relativeVolatilityInput.disabled = true;
            runSimulation();
        };

    </script>
</body>
</html>

