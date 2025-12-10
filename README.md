# Calculadora-Circuito-
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Calculadora de Circuito RL</title>
    <style>
        body { background: white; font-family: Arial, sans-serif; margin: 20px; color: #333; }
        h1 { text-align: center; color: #007BFF; font-size: 2.2em; }
        .container { max-width: 900px; margin: 0 auto; }
        .input-group { margin: 20px 0; display: flex; align-items: center; flex-wrap: wrap; gap: 15px; }
        label { font-weight: bold; width: 280px; }
        input { padding: 10px; width: 200px; border: 1px solid #007BFF; border-radius: 5px; }
        .preview { font-size: 1.1em; padding: 10px; background: #f0f8ff; border-radius: 5px; min-height: 50px; }
        button { background: #007BFF; color: white; padding: 12px 30px; border: none; border-radius: 5px; cursor: pointer; font-size: 1.1em; }
        button:hover { background: #0056b3; }
        #results { margin-top: 40px; display: none; }
        .graph { width: 48%; margin: 20px 1%; }
        .formula-box { background: #f9f9ff; padding: 15px; border-left: 5px solid #007BFF; margin-top: 10px; }
        table { width: 100%; border-collapse: collapse; margin-top: 20px; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: center; }
        th { background: #f0f8ff; }
        .optional { color: #666; font-style: italic; }
        @media (max-width: 768px) { .graph { width: 100%; } }
    </style>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
<div class="container">
    <h1>Calculadora de Circuito RL</h1>
    <p style="text-align:center;">Simulador educativo del decaimiento de corriente en un circuito RL serie<br>
       (Ecuación diferencial de primer orden homogénea)</p>

    <div class="input-group">
        <label>Resistencia R (ohm o kΩ):</label>
        <input type="number" id="R" step="any" placeholder="Ej: 1000 o 1k" oninput="updateResistor()">
        <div class="preview" id="resistorColors">Ingresa un valor →</div>
    </div>

    <div class="input-group">
        <label>Inductancia L (henry, mH o μH):</label>
        <input type="number" id="L" step="any" placeholder="Ej: 0.5 o 500m" oninput="updateInductor()">
        <div class="preview" id="inductorInfo">Ingresa un valor →</div>
    </div>

    <div class="input-group">
        <label>Corriente inicial I₀ (A):</label>
        <input type="number" id="I0" step="any" required>
    </div>

    <div class="input-group">
        <label>Tiempo máximo t_max (s):</label>
        <input type="number" id="tmax" step="any" required>
    </div>

    <div class="input-group">
        <label>Paso de tiempo Δt (s):</label>
        <input type="number" id="dt" step="any" required>
    </div>

    <div class="input-group optional">
        <label>Corriente objetivo I_objetivo (A):</label>
        <input type="number" id="Iobj" step="any">
    </div>

    <div class="input-group optional">
        <label>t_medido (s) e I_medido (A):</label>
        <input type="number" id="tmed" step="any" placeholder="t_medido">
        <input type="number" id="Imed" step="any" placeholder="I_medido">
    </div>

    <center><button onclick="calcular()">Calcular</button></center>
</div>

<div id="results" class="container">
    <h2>Resultados</h2>
    <div id="summary"></div>
    <p id="tau"></p>
    <p id="funcion"></p>
    <p id="valoresClave"></p>

    <div style="display:flex;flex-wrap:wrap;">
        <div class="graph"><canvas id="graficaI"></canvas>
            <div class="formula-box" id="formulaI"></div></div>
        <div class="graph"><canvas id="graficaVR"></canvas>
            <div class="formula-box" id="formulaVR"></div></div>
        <div class="graph"><canvas id="graficaVL"></canvas>
            <div class="formula-box" id="formulaVL"></div></div>
        <div class="graph"><canvas id="graficaTotal"></canvas>
            <div class="formula-box" id="formulaTotal"></div></div>
    </div>

    <h3>Tabla de valores</h3>
    <table id="tabla"></table>
</div>

<script>
const colores = ["black","brown","red","orange","yellow","green","blue","violet","grey","white"];
const tolerancias = ["","±1%","±2%","±0.5%","±0.25%","±0.10%","±0.05%","±5%","±10%"];

function updateResistor() {
    let val = parseFloat(document.getElementById("R").value);
    if (isNaN(val) || val <= 0) { document.getElementById("resistorColors").innerHTML = "Ingresa un valor > 0"; return; }
    
    // Conversión automática kΩ → Ω
    if (document.getElementById("R").value.includes("k")) val *= 1000;
    
    let valor = val;
    let exp = 0;
    while (valor >= 100) { valor /= 10; exp++; }
    valor = Math.round(valor * 100) / 100;

    let dig1 = Math.floor(valor / 10);
    let dig2 = Math.floor(valor % 10);
    let mult = exp;
    let tol = 5; // oro = ±5% (más común)

    document.getElementById("resistorColors").innerHTML = `
        <strong>Resistencia ≈ ${val.toLocaleString()} Ω</strong><br>
        Bandas: <span style="color:${colores[dig1]}">■</span> 
                <span style="color:${colores[dig2]}">■</span> 
                <span style="color:${colores[mult]}">■</span> 
                <span style="color:gold">■</span> → 
        ${dig1}${dig2} × 10<sup>${mult}</sup> Ω ${tolerancias[tol]}<br>
        <small>Ejemplo visual de resistencia de 4 bandas</small>`;
}

function updateInductor() {
    let val = parseFloat(document.getElementById("L").value);
    if (isNaN(val) || val <= 0) { document.getElementById("inductorInfo").innerHTML = "Ingresa un valor > 0"; return; }
    
    let Lh = val;
    if (document.getElementById("L").value.includes("m")) Lh /= 1000;
    if (document.getElementById("L").value.includes("u") || document.getElementById("L").value.includes("μ")) Lh /= 1e6;

    let vueltas = Math.round(300 * Math.pow(Lh, 0.4));
    let diametro = Lh < 0.01 ? "pequeño (5-10 mm)" : Lh < 0.1 ? "mediano (10-30 mm)" : "grande (>30 mm)";

    document.getElementById("inductorInfo").innerHTML = `
        <strong>Bobina ≈ ${Lh.toFixed(6)} H</strong><br>
        Estimación didáctica:<br>
        • ~${vueltas} vueltas de alambre<br>
        • Diámetro aproximado: ${diametro}<br>
        <small>Valores típicos en bobinas reales</small>`;
}

function calcular() {
    // Lectura y conversión automática
    let R = parseFloat(document.getElementById("R").value) || 0;
    if (document.getElementById("R").value.includes("k")) R *= 1000;

    let L = parseFloat(document.getElementById("L").value) || 0;
    if (document.getElementById("L").value.includes("m")) L /= 1000;
    if (document.getElementById("L").value.includes("u") || document.getElementById("L").value.includes("μ")) L /= 1e6;

    const I0 = parseFloat(document.getElementById("I0").value);
    const tmax = parseFloat(document.getElementById("tmax").value);
    const dt = parseFloat(document.getElementById("dt").value);
    const Iobj = parseFloat(document.getElementById("Iobj").value) || null;
    const tmed = parseFloat(document.getElementById("tmed").value) || null;
    const Imed = parseFloat(document.getElementById("Imed").value) || null;

    if (R <= 0 || L <= 0 || !I0 || !tmax || !dt) { alert("Complete todos los campos obligatorios con valores positivos"); return; }

    const tau = L / R;

    // Resumen
    document.getElementById("summary").innerHTML = `<h3>Resumen de datos</h3>
        R = ${R.toFixed(2)} Ω  L = ${L.toFixed(6)} H  I₀ = ${I0} A  t_max = ${tmax} s  Δt = ${dt} s`;

    document.getElementById("tau").innerHTML = `<strong>Constante de tiempo τ = L/R = ${tau.toFixed(6)} s</strong><br>
        Es el tiempo característico del decaimiento: a t = τ la corriente cae al 36.8% de I₀`;

    document.getElementById("funcion").innerHTML = `<strong>i(t) = ${I0} × e<sup>-t / ${tau.toFixed(6)}</sup> A</strong>`;

    // Cálculos clave
    let clave = `• i(0) = ${I0} A  • i(t_max) = ${(I0 * Math.exp(-tmax/tau)).toFixed(4)} A`;
    if (Iobj !== null) {
        const tObj = -tau * Math.log(Iobj / I0);
        clave += `<br>• Tiempo para I_obj = ${Iobj} A → t = ${tObj.toFixed(4)} s`;
    }
    if (tmed !== null && Imed !== null) {
        const Icalc = I0 * Math.exp(-tmed/tau);
        const error = Math.abs((Imed - Icalc)/Icalc)*100;
        clave += `<br>• A t = ${tmed} s → i(teórico) = ${Icalc.toFixed(4)} A  Error = ${error.toFixed(2)}%`;
    }
    document.getElementById("valoresClave").innerHTML = "<h3>Resultados clave</h3>" + clave;

    // Tabla y gráficas
    const tiempos = [], corrientes = [], vR = [], vL = [];
    for (let t = 0; t <= tmax; t += dt) {
        tiempos.push(t.toFixed(4));
        const i = I0 * Math.exp(-t / tau);
        corrientes.push(i.toFixed(4));
        vR.push((i * R).toFixed(4));
        vL.push((-i * R).toFixed(4));
    }

    // Tabla
    let tabla = `<thead><tr><th>t (s)</th><th>i(t) (A)</th><th>v_R(t) (V)</th><th>v_L(t) (V)</th></tr></thead><tbody>`;
    tiempos.forEach((t, i) => {
        tabla += `<tr><td>${t}</td><td>${corrientes[i]}</td><td>${vR[i]}</td><td>${vL[i]}</td></tr>`;
    });
    document.getElementById("tabla").innerHTML = tabla + "</tbody>";

    // Gráficas
    crearGrafica("graficaI", tiempos, corrientes, "Corriente i(t)", "#007BFF");
    crearGrafica("graficaVR", tiempos, vR, "Voltaje en R", "#D32F2F");
    crearGrafica("graficaVL", tiempos, vL, "Voltaje en L", "#D32F2F");
    crearGrafica("graficaTotal", tiempos, new Array(tiempos.length).fill(0), "Voltaje total = 0 V", "#000000");

    // Fórmulas debajo de cada gráfica
    document.getElementById("formulaI").innerHTML = `<strong>i(t) = ${I0} e<sup>-t/${tau.toFixed(6)}</sup></strong><br>Ej: t=0 → i=${I0} A`;
    document.getElementById("formulaVR").innerHTML = `<strong>v_R(t) = ${I0}×${R} e<sup>-t/${tau.toFixed(6)}</sup> = ${(I0*R).toFixed(2)} e<sup>-t/${tau.toFixed(6)}</sup> V</strong>`;
    document.getElementById("formulaVL").innerHTML = `<strong>v_L(t) = -${I0}×${R} e<sup>-t/${tau.toFixed(6)}</sup> = ${(-I0*R).toFixed(2)} e<sup>-t/${tau.toFixed(6)}</sup> V</strong>`;
    document.getElementById("formulaTotal").innerHTML = `<strong>v_R(t) + v_L(t) = 0 V (Ley de Kirchhoff)</strong>`;

    document.getElementById("results").style.display = "block";
    window.scrollTo(0, document.getElementById("results").offsetTop);
}

function crearGrafica(id, labels, data, titulo, color) {
    new Chart(document.getElementById(id), {
        type: 'line',
        data: { labels: labels, datasets: [{ label: titulo, data: data, borderColor: color, fill: false, tension: 0.1 }] },
        options: { responsive: true, scales: { x: { title: { display: true, text: 'Tiempo (s)' }}}}
    });
}
</script>
</body>
</html>
