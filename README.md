<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>LED Panel Calculator</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #121212;
            color: #e0e0e0;
            margin: 20px;
        }
        .container {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            border: 1px solid #333;
            border-radius: 10px;
            background-color: #1e1e1e;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
        }
        input, button, select {
            width: 100%;
            padding: 10px;
            margin-top: 10px;
            font-size: 16px;
            border: none;
            border-radius: 5px;
        }
        input, select {
            background-color: #2c2c2c;
            color: #e0e0e0;
        }
        input::placeholder {
            color: #888;
        }
        button {
            background-color: #ff9800;
            color: #fff;
            font-weight: bold;
            cursor: pointer;
            transition: background-color 0.3s;
        }
        button:hover {
            background-color: #e68900;
        }
        .result {
            margin-top: 20px;
            padding: 10px;
            background-color: #2c2c2c;
            border: 1px solid #444;
            border-radius: 5px;
        }
        .quantity-buttons {
            display: flex;
            gap: 5px;
            align-items: center;
            margin-top: 5px;
        }
        .quantity-buttons button {
            width: 30px;
            height: 30px;
            font-size: 14px;
            padding: 0;
            background-color: #ff9800;
            color: #fff;
            border: none;
            border-radius: 3px;
            cursor: pointer;
            transition: background-color 0.3s;
        }
        .quantity-buttons button:hover {
            background-color: #e68900;
        }
    </style>
</head>
<body>
<div class="container">
    <h1>LED Panel Calculator</h1>
    <label for="area">Enter Area (m²):</label>
    <input type="number" id="area" placeholder="Enter area in square meters" min="0" step="0.01" />

    <label for="model">Select Model:</label>
    <select id="model">
        <option value="3.9 Outdoor">3.9 Outdoor</option>
        <option value="3.9 Indoor">3.9 Indoor</option>
        <option value="2.9 Indoor">2.9 Indoor</option>
        <option value="2.6 Indoor">2.6 Indoor</option>
    </select>

    <button onclick="calculatePanels()">Calculate</button>

    <div class="result" id="result"></div>
</div>

<script>
    function calculatePanels() {
        const area = parseFloat(document.getElementById('area').value);
        const model = document.getElementById('model').value;

        if (isNaN(area) || area <= 0) {
            document.getElementById('result').textContent = 'Please enter a valid area.';
            return;
        }

        const models = {
            '3.9 Outdoor': { pixelDensity: 65536, weight: 32, power: 400 },
            '3.9 Indoor': { pixelDensity: 65536, weight: 30, power: 380 },
            '2.9 Indoor': { pixelDensity: 112896, weight: 35, power: 450 },
            '2.6 Indoor': { pixelDensity: 147456, weight: 38, power: 480 }
        };

        const specs = models[model];

        const panelArea500x1000 = 0.5 * 1.0;
        const panelArea500x500 = 0.5 * 0.5;

        const totalPanels500x1000 = Math.ceil(area / panelArea500x1000);
        let remainingPanels500x1000 = totalPanels500x1000;

        let rectPanels = Math.ceil(totalPanels500x1000 * 0.15);
        let curvePanels = Math.ceil(totalPanels500x1000 * 0.09);
        let bevelPanels = Math.ceil(totalPanels500x1000 * 0.06);

        remainingPanels500x1000 -= (rectPanels + curvePanels + bevelPanels);
        if (remainingPanels500x1000 < 0) remainingPanels500x1000 = 0;

        const totalPixels = area * specs.pixelDensity;
        const totalPower = (remainingPanels500x1000 + rectPanels + curvePanels + bevelPanels) * specs.power;
        const totalWeight = (remainingPanels500x1000 + rectPanels + curvePanels + bevelPanels) * specs.weight;
        const totalCases = Math.ceil(remainingPanels500x1000 / 6) + Math.ceil(rectPanels / 8) + Math.ceil(curvePanels / 8) + Math.ceil(bevelPanels / 8);
        const volume = totalCases * 0.5; // Assuming each case is 0.5m³

        // Cálculo salidas procesador
        const pixelsPerOutput = 650000;
        const processorOutputs = Math.ceil(totalPixels / pixelsPerOutput);

        const updateUI = () => {
            document.getElementById('result').innerHTML = `
                <strong>${model} Results:</strong><br>
                - Standard Panels (500x1000): <span id="panelCount500x1000">${remainingPanels500x1000}</span> (Cases: ${Math.ceil(remainingPanels500x1000 / 6)})
                <div class="quantity-buttons">
                    <button onclick="adjustQuantity('subtract', 'panelCount500x1000', '500x1000')">-</button>
                    <button onclick="adjustQuantity('add', 'panelCount500x1000', '500x1000')">+</button>
                </div>
                - Rect Panels (500x500): <span id="rectPanels">${rectPanels}</span> (Cases: ${Math.ceil(rectPanels / 8)})
                <div class="quantity-buttons">
                    <button onclick="adjustQuantity('subtract', 'rectPanels', '500x500')">-</button>
                    <button onclick="adjustQuantity('add', 'rectPanels', '500x500')">+</button>
                </div>
                - Curve Panels (500x500): <span id="curvePanels">${curvePanels}</span> (Cases: ${Math.ceil(curvePanels / 8)})
                <div class="quantity-buttons">
                    <button onclick="adjustQuantity('subtract', 'curvePanels', '500x500')">-</button>
                    <button onclick="adjustQuantity('add', 'curvePanels', '500x500')">+</button>
                </div>
                - Beveled Panels (500x500): <span id="bevelPanels">${bevelPanels}</span> (Cases: ${Math.ceil(bevelPanels / 8)})
                <div class="quantity-buttons">
                    <button onclick="adjustQuantity('subtract', 'bevelPanels', '500x500')">-</button>
                    <button onclick="adjustQuantity('add', 'bevelPanels', '500x500')">+</button>
                </div>
                <br>
                <strong>Total Calculations:</strong><br>
                - Total Pixels: ${totalPixels.toLocaleString()} px<br>
                - Total Power: ${totalPower.toLocaleString()} W<br>
                - Total Weight: ${totalWeight.toLocaleString()} kg<br>
                - Total Volume: ${volume.toFixed(2)} m³<br>
                - Processor Outputs Needed: ${processorOutputs}<br>
            `;
        };

        const adjustQuantity = (action, elementId, type) => {
            const element = document.getElementById(elementId);
            let currentCount = parseInt(element.textContent);

            if (action === 'add') {
                currentCount++;
                if (type === '500x500') {
                    remainingPanels500x1000--;
                    if (remainingPanels500x1000 < 0) {
                        remainingPanels500x1000 = 0;
                    }
                }
            } else if (action === 'subtract' && currentCount > 0) {
                currentCount--;
                if (type === '500x500') {
                    remainingPanels500x1000++;
                }
            }

            if (type === '500x500') {
                if (elementId === 'rectPanels') rectPanels = currentCount;
                if (elementId === 'curvePanels') curvePanels = currentCount;
                if (elementId === 'bevelPanels') bevelPanels = currentCount;
            }

            element.textContent = currentCount;
            updateUI();
        };

        window.adjustQuantity = adjustQuantity;
        updateUI();
    }
</script>
</body>
</html>
