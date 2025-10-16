# simulador-destilacion
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
        .control-panel {
            background-color: #1a202c; /* Un gris oscuro para el panel */
        }
        .tower-container {
            background-color: #2d3748; /* Un gris ligeramente más claro para el fondo de la torre */
        }
        .btn-run {
            background-color: #4299e1; /* Azul para el botón de simulación */
        }
        .btn-run:hover {
            background-color: #2b6cb0;
        }
        .results-card {
            background-color: #4a5568;
        }
    </style>
</head>
<body class="bg-gray-900 text-white flex flex-col items-center justify-center min-h-screen p-4">

    <div class="w-full max-w-6xl mx-auto">
        <h1 class="text-4xl font-bold text-center mb-2 text-blue-300">Simulador de Torre de Destilación</h1>
        <p class="text-center text-gray-400 mb-8">Una herramienta para visualizar la destilación binaria basada en el método McCabe-Thiele.</p>

        <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">

            <!-- Panel de Control -->
            <div class="lg:col-span-1 control-panel p-6 rounded-xl shadow-lg">
                <h2 class="text-2xl font-semibold mb-6 border-b border-gray-600 pb-2">Parámetros de Entrada</h2>

                <div class="space-y-6">
                    <div>
                        <label for="feedComposition" class="block mb-2 text-sm font-medium text-gray-300">Composición de Alimentación (Fracción Molar Comp. Ligero): <span id="feedCompositionValue" class="font-bold text-blue-300">0.50</span></label>
                        <input id="feedComposition" type="range" min="0.01" max="0.99" value="0.5" step="0.01" class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer">
                    </div>
                    <div>
                        <label for="refluxRatio" class="block mb-2 text-sm font-medium text-gray-300">Relación de Reflujo (R): <span id="refluxRatioValue" class="font-bold text-blue-300">1.5</span></label>
                        <input id="refluxRatio" type="range" min="0.5" max="5" value="1.5" step="0.1" class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer">
                    </div>
                    <div>
                        <label for="relativeVolatility" class="block mb-2 text-sm font-medium text-gray-300">Volatilidad Relativa (α): <span id="relativeVolatilityValue" class="font-bold text-blue-300">2.5</span></label>
                        <input id="relativeVolatility" type="range" min="1.1" max="5" value="2.5" step="0.1" class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer">
                    </div>
                     <div>
                        <label for="qValue" class="block mb-2 text-sm font-medium text-gray-300">Calidad de la Alimentación (q): <span id="qValueDisplay" class="font-bold text-blue-300">1.0</span></label>
                        <input id="qValue" type="range" min="0" max="1.5" value="1.0" step="0.1" class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer">
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
                <div class="tower-container p-4 rounded-xl shadow-lg h-full">
                    <h2 class="text-xl font-semibold mb-4 text-center">Diagrama de Equilibrio y Etapas</h2>
                    <canvas id="mccabeThieleCanvas"></canvas>
                </div>

                <!-- Resultados -->
                <div class="results-card p-6 rounded-xl shadow-lg">
                    <h2 class="text-2xl font-semibold mb-4 border-b border-gray-600 pb-2">Resultados de la Simulación</h2>
                    <div id="resultsOutput" class="grid grid-cols-1 sm:grid-cols-3 gap-4 text-center">
                        <div>
                            <h3 class="text-lg font-medium text-blue-300">Etapas Teóricas</h3>
                            <p id="theoreticalStages" class="text-3xl font-bold">--</p>
                        </div>
                        <div>
                            <h3 class="text-lg font-medium text-green-300">Composición Destilado (x_D)</h3>
                            <p id="distillateComposition" class="text-3xl font-bold">--</p>
                        </div>
                        <div>
                            <h3 class="text-lg font-medium text-yellow-300">Composición Fondos (x_B)</h3>
                            <p id="bottomsComposition" class="text-3xl font-bold">--</p>
                        </div>
                    </div>
                     <div id="message" class="text-center mt-4 text-red-400 font-medium"></div>
                </div>
            </div>

        </div>
    </div>

    <script>
        // Elementos del DOM
        const feedCompositionSlider = document.getElementById('feedComposition');
        const refluxRatioSlider = document.getElementById('refluxRatio');
        const relativeVolatilitySlider = document.getElementById('relativeVolatility');
        const qValueSlider = document.getElementById('qValue');
        const runButton = document.getElementById('runSimulation');
        const canvas = document.getElementById('mccabeThieleCanvas');
        const ctx = canvas.getContext('2d');

        // Displays de valores
        const feedCompositionValue = document.getElementById('feedCompositionValue');
        const refluxRatioValue = document.getElementById('refluxRatioValue');
        const relativeVolatilityValue = document.getElementById('relativeVolatilityValue');
        const qValueDisplay = document.getElementById('qValueDisplay');

        // Salidas de resultados
        const theoreticalStagesEl = document.getElementById('theoreticalStages');
        const distillateCompositionEl = document.getElementById('distillateComposition');
        const bottomsCompositionEl = document.getElementById('bottomsComposition');
        const messageEl = document.getElementById('message');

        // Configuración inicial
        const desiredDistillateComp = 0.95;
        const desiredBottomsComp = 0.05;

        // Actualizar displays de los sliders
        feedCompositionSlider.addEventListener('input', () => feedCompositionValue.textContent = parseFloat(feedCompositionSlider.value).toFixed(2));
        refluxRatioSlider.addEventListener('input', () => refluxRatioValue.textContent = parseFloat(refluxRatioSlider.value).toFixed(1));
        relativeVolatilitySlider.addEventListener('input', () => relativeVolatilityValue.textContent = parseFloat(relativeVolatilitySlider.value).toFixed(1));
        qValueSlider.addEventListener('input', () => qValueDisplay.textContent = parseFloat(qValueSlider.value).toFixed(1));
        
        // Función para dibujar el gráfico base
        function drawBaseChart() {
            const parentEl = canvas.parentElement;
            if (!parentEl) return; // Defensive check

            const width = parentEl.clientWidth * 0.95;
            const height = width * 0.8; // Mantener relación de aspecto
            canvas.width = width;
            canvas.height = height;

            const padding = 50;
            const chartWidth = width - 2 * padding;
            const chartHeight = height - 2 * padding;

            ctx.clearRect(0, 0, width, height);

            // Fondo
            ctx.fillStyle = '#1f2937';
            ctx.fillRect(0, 0, width, height);

            ctx.save();
            ctx.translate(padding, padding);

            // Ejes
            ctx.strokeStyle = '#9ca3af';
            ctx.lineWidth = 1;
            ctx.beginPath();
            ctx.moveTo(0, 0);
            ctx.lineTo(0, chartHeight);
            ctx.lineTo(chartWidth, chartHeight);
            ctx.stroke();

            // Etiquetas de ejes
            ctx.fillStyle = '#d1d5db';
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
                // Eje X
                const x = (i / 10) * chartWidth;
                ctx.fillText(val, x, chartHeight + 15);
                ctx.beginPath();
                ctx.moveTo(x, chartHeight);
                ctx.lineTo(x, chartHeight - 5);
                ctx.stroke();

                // Eje Y
                const y = chartHeight - (i / 10) * chartHeight;
                ctx.fillText(val, -20, y);
                ctx.beginPath();
                ctx.moveTo(0, y);
                ctx.lineTo(5, y);
                ctx.stroke();
            }

            // Línea y=x
            ctx.strokeStyle = '#4b5563';
            ctx.setLineDash([5, 5]);
            ctx.beginPath();
            ctx.moveTo(0, chartHeight);
            ctx.lineTo(chartWidth, 0);
            ctx.stroke();
            ctx.setLineDash([]);
            
            ctx.restore();
        }

        // Función de simulación
        function runSimulation() {
            messageEl.textContent = '';
            drawBaseChart();

            const zF = parseFloat(feedCompositionSlider.value);
            const R = parseFloat(refluxRatioSlider.value);
            const alpha = parseFloat(relativeVolatilitySlider.value);
            const q = parseFloat(qValueSlider.value);
            const xD = desiredDistillateComp;
            const xB = desiredBottomsComp;

            const width = canvas.width;
            const height = canvas.height;
            const padding = 50;
            const chartWidth = width - 2 * padding;
            const chartHeight = height - 2 * padding;
            
            ctx.save();
            ctx.translate(padding, padding);

            // Escala de coordenadas
            const toCanvasX = (x) => x * chartWidth;
            const toCanvasY = (y) => chartHeight - y * chartHeight;

            // 1. Dibujar curva de equilibrio
            ctx.strokeStyle = '#60a5fa'; // Azul brillante
            ctx.lineWidth = 2;
            ctx.beginPath();
            for (let i = 0; i <= 100; i++) {
                const x = i / 100;
                const y = (alpha * x) / (1 + (alpha - 1) * x);
                if (i === 0) {
                    ctx.moveTo(toCanvasX(x), toCanvasY(y));
                } else {
                    ctx.lineTo(toCanvasX(x), toCanvasY(y));
                }
            }
            ctx.stroke();

            // 2. Dibujar línea q (línea de alimentación)
            let qSlope, qIntercept;
            if (q === 1) { // Líquido saturado (vertical)
                qSlope = Infinity;
            } else {
                qSlope = q / (q - 1);
            }
            qIntercept = zF * (1 - qSlope);

            let startX_q = 0, startY_q = 0, endX_q = 1, endY_q = 1;
            if (q === 1) {
                 startX_q = zF; endX_q = zF;
                 startY_q = zF; endY_q = (alpha * zF) / (1 + (alpha-1)*zF);
            } else {
                const intersectY_at_x0 = qIntercept;
                const intersectX_at_y1 = (1 - qIntercept) / qSlope;
                
                startX_q = 0; startY_q = intersectY_at_x0;
                if(intersectY_at_x0 < 0) {
                    startX_q = -qIntercept / qSlope; startY_q = 0;
                }
                
                endX_q = 1; endY_q = qSlope + qIntercept;
                 if (endY_q > 1) {
                    endX_q = intersectX_at_y1; endY_q = 1;
                }
            }
            ctx.strokeStyle = '#facc15'; // Amarillo
            ctx.beginPath();
            ctx.moveTo(toCanvasX(zF), toCanvasY(zF));
            const y_intersect_eq = (alpha * zF) / (1 + (alpha-1)*zF);
            if (q === 1) {
                ctx.lineTo(toCanvasX(zF), toCanvasY(y_intersect_eq));
            } else {
                 const x_intersect_diag = zF;
                 const y_intersect_diag = zF;
                 const y_eq = (alpha*x_intersect_diag) / (1 + (alpha-1)*x_intersect_diag);

                 let x_end_q = (y_eq - qIntercept) / qSlope;
                 ctx.lineTo(toCanvasX(x_end_q), toCanvasY(y_eq));
            }
            ctx.stroke();
            
            // 3. Dibujar línea de operación de enriquecimiento (rectifying)
            const rect_slope = R / (R + 1);
            const rect_intercept = xD / (R + 1);
            
            ctx.strokeStyle = '#4ade80'; // Verde
            ctx.beginPath();
            ctx.moveTo(toCanvasX(xD), toCanvasY(xD));
            
            // Encontrar punto de intersección entre la línea q y la de enriquecimiento
            let intersectX, intersectY;
             if (q === 1) {
                intersectX = zF;
                intersectY = rect_slope * zF + rect_intercept;
            } else {
                intersectX = (rect_intercept - qIntercept) / (qSlope - rect_slope);
                intersectY = rect_slope * intersectX + rect_intercept;
            }
            ctx.lineTo(toCanvasX(intersectX), toCanvasY(intersectY));
            ctx.stroke();
            
            // 4. Dibujar línea de operación de agotamiento (stripping)
            ctx.strokeStyle = '#f87171'; // Rojo
            ctx.beginPath();
            ctx.moveTo(toCanvasX(xB), toCanvasY(xB));
            ctx.lineTo(toCanvasX(intersectX), toCanvasY(intersectY));
            ctx.stroke();

            // 5. Dibujar etapas
            let stages = 0;
            let currentX = xD;
            let currentY = xD;

            ctx.strokeStyle = '#a78bfa'; // Violeta
            ctx.lineWidth = 1;

            if (rect_intercept <= 0) {
                 messageEl.textContent = 'Condición de reflujo mínimo o por debajo. Aumente la Relación de Reflujo.';
                 theoreticalStagesEl.textContent = '∞';
                 distillateCompositionEl.textContent = desiredDistillateComp.toFixed(2);
                 bottomsCompositionEl.textContent = desiredBottomsComp.toFixed(2);
                 ctx.restore();
                 return;
            }

            while (currentX > xB && stages < 50) {
                // Línea horizontal (del vapor al equilibrio)
                const equilibriumX = findEquilibriumX(currentY, alpha);
                ctx.beginPath();
                ctx.moveTo(toCanvasX(currentX), toCanvasY(currentY));
                ctx.lineTo(toCanvasX(equilibriumX), toCanvasY(currentY));
                ctx.stroke();
                
                currentX = equilibriumX;
                
                // Línea vertical (del líquido a la línea de operación)
                let nextY;
                if (currentX > intersectX) {
                    // Zona de enriquecimiento
                    nextY = rect_slope * currentX + rect_intercept;
                } else {
                    // Zona de agotamiento
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

            // Actualizar resultados
            theoreticalStagesEl.textContent = stages;
            distillateCompositionEl.textContent = xD.toFixed(2);
            bottomsCompositionEl.textContent = xB.toFixed(2);
        }
        
        // Función para encontrar X en la curva de equilibrio dado Y
        function findEquilibriumX(y, alpha) {
            return y / (alpha - y * (alpha - 1));
        }

        // Event Listeners
        runButton.addEventListener('click', runSimulation);
        window.addEventListener('resize', drawBaseChart);

        // Simulación inicial al cargar la página
        window.onload = () => {
             // Forzar el disparo del evento input para que los valores se muestren
             feedCompositionSlider.dispatchEvent(new Event('input'));
             refluxRatioSlider.dispatchEvent(new Event('input'));
             relativeVolatilitySlider.dispatchEvent(new Event('input'));
             qValueSlider.dispatchEvent(new Event('input'));
             runSimulation();
        };

    </script>
</body>
</html>

