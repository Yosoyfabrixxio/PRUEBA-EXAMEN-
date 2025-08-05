<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Resolución de Problemas con Método Simplex</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/mathjs/10.6.4/math.min.js"></script>
  <style>
    body { font-family: Arial, sans-serif; background-color: #f4f4f4; padding: 20px; }
    textarea, button { width: 100%; margin-top: 10px; padding: 12px; font-size: 1em; }
    #resultado, #tabla, #grafico { background: #fff; padding: 20px; margin-top: 20px; border-radius: 10px; box-shadow: 0 0 10px #ccc; }
    table { width: 100%; border-collapse: collapse; margin-top: 15px; }
    th, td { border: 1px solid #aaa; padding: 8px; text-align: center; }
    h2, h3 { color: #333; }
    canvas { width: 100% !important; height: auto !important; }
  </style>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
  <h2>📌 Resolución Automática de Problemas con Método Simplex</h2>
  <textarea id="problema" rows="12" placeholder="Pega aquí el enunciado completo del problema..."></textarea>
  <button onclick="resolverProblema()">Interpretar y Resolver</button>

  <div id="resultado"></div>
  <div id="tabla"></div>
  <div id="grafico"><canvas id="graficoCanvas"></canvas></div>

<script>
function resolverProblema() {
  const texto = document.getElementById("problema").value.toLowerCase();
  let resultadoHTML = "", tablaHTML = "";

  if (texto.includes("tucker")) {
    resolverTucker();
  } else if (texto.includes("investment advisors")) {
    resolverInvestmentAdvisors();
  } else {
    document.getElementById("resultado").innerHTML = `<p style='color:red;'>Este problema aún no es reconocido. Prueba con el ejercicio de Tucker Inc. o Investment Advisors.</p>`;
    document.getElementById("tabla").innerHTML = "";
  }
}

function resolverInvestmentAdvisors() {
  const variables = ['U', 'H'];
  const restricciones = [
    { texto: '25U + 50H <= 80000', coef: [25, 50], signo: '<=', valor: 80000 },
    { texto: '0.50U + 0.25H <= 700', coef: [0.5, 0.25], signo: '<=', valor: 700 },
    { texto: 'U <= 1000', coef: [1, 0], signo: '<=', valor: 1000 }
  ];
  const objFunc = { tipo: 'max', coef: [3, 5] };

  let factibles = [];
  for (let u = 0; u <= 1000; u++) {
    for (let h = 0; h <= 1000; h++) {
      let esFactible = true;
      for (let r of restricciones) {
        const z = r.coef[0] * u + r.coef[1] * h;
        if ((r.signo === '<=' && z > r.valor) ||
            (r.signo === '>=' && z < r.valor)) {
          esFactible = false;
          break;
        }
      }
      if (esFactible) {
        const rendimiento = objFunc.coef[0] * u + objFunc.coef[1] * h;
        factibles.push({ u, h, rendimiento });
      }
    }
  }

  if (factibles.length === 0) {
    document.getElementById("resultado").innerHTML = `<p style='color:red;'>No se encontró una región factible. Revisa las restricciones o el rango de búsqueda.</p>`;
    return;
  }

  factibles.sort((a, b) => b.rendimiento - a.rendimiento);
  const optimo = factibles[0];

  let resultadoHTML = `<h3>✅ Solución Óptima</h3>`;
  resultadoHTML += `<p><strong>Acciones U.S. Oil (U):</strong> ${optimo.u} &nbsp; | &nbsp; <strong>Acciones Huber Steel (H):</strong> ${optimo.h}</p>`;
  resultadoHTML += `<p><strong>Rendimiento Máximo:</strong> $${optimo.rendimiento}</p>`;

  let tablaHTML = `<h3>📊 Tabla de Evaluación (Top 20)</h3>`;
  tablaHTML += `<table><tr><th>U</th><th>H</th><th>Rendimiento</th></tr>`;
  factibles.slice(0, 20).forEach(p => {
    tablaHTML += `<tr><td>${p.u}</td><td>${p.h}</td><td>${p.rendimiento}</td></tr>`;
  });
  tablaHTML += `</table>`;

  resultadoHTML += `<h3>📌 Análisis Profesional</h3>`;
  resultadoHTML += `<p>La solución óptima sugiere comprar <strong>${optimo.u}</strong> acciones de U.S. Oil y <strong>${optimo.h}</strong> de Huber Steel para maximizar el rendimiento total a <strong>$${optimo.rendimiento}</strong>, cumpliendo las restricciones de inversión, riesgo y límite de acciones.</p>`;

  const ctx = document.getElementById('graficoCanvas').getContext('2d');
  const puntos = factibles.slice(0, 100);
  const data = {
    datasets: [{
      label: 'Soluciones factibles',
      data: puntos.map(p => ({ x: p.u, y: p.h })),
      backgroundColor: 'rgba(54, 162, 235, 0.5)',
      pointRadius: 3
    }, {
      label: 'Óptimo',
      data: [{ x: optimo.u, y: optimo.h }],
      backgroundColor: 'red',
      pointRadius: 6
    }]
  };

  new Chart(ctx, {
    type: 'scatter',
    data: data,
    options: {
      plugins: {
        legend: { position: 'top' }
      },
      scales: {
        x: {
          title: { display: true, text: 'U.S. Oil (U)' },
          grid: { color: '#ddd' },
          ticks: { stepSize: 50 }
        },
        y: {
          title: { display: true, text: 'Huber Steel (H)' },
          grid: { color: '#ddd' },
          ticks: { stepSize: 50 }
        }
      }
    }
  });

  document.getElementById("resultado").innerHTML = resultadoHTML;
  document.getElementById("tabla").innerHTML = tablaHTML;
}
</script>
</body>
</html>
