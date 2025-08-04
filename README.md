<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Simplex Interpretador Mejorado</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      padding: 20px;
      background-color: #f0f2f5;
      color: #333;
      line-height: 1.6;
    }
    .container {
      max-width: 900px;
      margin: 0 auto;
      background: #fff;
      padding: 30px;
      border-radius: 8px;
      box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    }
    h2 {
      text-align: center;
      color: #0056b3;
      margin-bottom: 20px;
    }
    textarea, button {
      width: 100%;
      margin: 10px 0;
      padding: 12px;
      border-radius: 5px;
      border: 1px solid #ccc;
      font-size: 16px;
      box-sizing: border-box; /* Asegura que padding no aumente el ancho */
    }
    button {
      background-color: #007bff;
      color: white;
      border: none;
      cursor: pointer;
      transition: background-color 0.3s ease;
    }
    button:hover {
      background-color: #0056b3;
    }
    canvas {
      background: #fff;
      margin-top: 20px;
      border: 1px solid #eee;
      border-radius: 5px;
    }
    table {
      border-collapse: collapse;
      width: 100%;
      margin-top: 20px;
      background: #fff;
      font-size: 14px;
    }
    th, td {
      border: 1px solid #ddd;
      padding: 12px;
      text-align: left;
    }
    th {
      background-color: #f4f4f4;
      font-weight: bold;
    }
    tr:nth-child(even) {
      background-color: #f9f9f9;
    }
    #resultado {
      margin-top: 20px;
      padding: 20px;
      background: #f9f9f9;
      border: 1px solid #e0e0e0;
      border-radius: 5px;
    }
    #resultado h3 {
      color: #28a745;
      margin-top: 0;
    }
    .error {
      color: #dc3545;
      font-weight: bold;
    }
  </style>
</head>
<body>

<div class="container">
  <h2>🔍 Resolver Problemas de Programación Lineal</h2>
  <textarea id="textoProblema" rows="8" placeholder="Pega aquí el enunciado del problema (ej. Maximizar Z = 20X + 30Y sujeto a 3X + 4Y <= 100 y 5X + 2Y <= 80)"></textarea>
  <button onclick="resolverProblema()">📌 Interpretar y Resolver</button>

  <div id="resultado"></div>
  <canvas id="grafica" width="900" height="600"></canvas>
</div>

<script>
// Usa una función de alto nivel para manejar toda la lógica y el manejo de errores
function resolverProblema() {
  const outputDiv = document.getElementById("resultado");
  outputDiv.innerHTML = ""; // Limpia el resultado anterior

  const textoOriginal = document.getElementById("textoProblema").value;
  if (!textoOriginal) {
    outputDiv.innerHTML = '<p class="error">Por favor, introduce un problema para resolver.</p>';
    return;
  }

  try {
    const texto = textoOriginal.toLowerCase().replace(/\n/g, " ");

    const isMax = !texto.includes('minimizar');
    
    // Mejor detección de variables para manejar múltiples letras
    const variables = [...new Set(texto.matchAll(/(\b[a-z]\b)/g))]
                      .map(match => match[0].toUpperCase())
                      .sort();

    if (variables.length < 2) {
      throw new Error("No se detectaron al menos dos variables (ej. 'x' y 'y').");
    }

    const [var1, var2] = variables;

    // --- PARSEO DE LA FUNCIÓN OBJETIVO ---
    let objStr = parseObjetivo(texto, var1, var2, isMax);

    // --- PARSEO DE RESTRICCIONES ---
    const restricciones = parseRestricciones(texto, var1, var2);
    restricciones.push(`${var1} >= 0`, `${var2} >= 0`);

    // --- RESOLUCIÓN Y VISUALIZACIÓN ---
    resolver(objStr, restricciones, var1, var2, isMax);

  } catch (error) {
    outputDiv.innerHTML = `<p class="error">Error: ${error.message}</p>`;
    console.error(error);
  }
}

function parseObjetivo(texto, var1, var2, isMax) {
  const matches = [...texto.matchAll(/(\d+(?:\.\d+)?)\s*([a-z])\b/g)];
  if (matches.length === 0) {
    throw new Error("No se pudo detectar la función objetivo. Asegúrate de que tenga la forma '20x + 30y'.");
  }

  let objExpr = matches.map(m => {
    const value = parseFloat(m[1]);
    const variable = m[2].toUpperCase();
    return `${value}${variable}`;
  }).join(' + ');

  if (!isMax) {
    // Si es minimización, invertimos los signos para maximizar el negativo
    const terms = objExpr.split(' + ').map(term => {
      const parts = term.match(/(\d+(?:\.\d+)?)([A-Z])/);
      return `-${parts[1]}${parts[2]}`;
    });
    objExpr = terms.join(' + ');
    return `Z = ${objExpr}`;
  }
  
  return `Z = ${objExpr}`;
}

function parseRestricciones(texto, var1, var2) {
  const restricciones = [];
  const lines = texto.split(/sujeto a|restricciones|y\b|además/);

  lines.forEach(line => {
    const match = line.match(/(?:(\d+(?:\.\d+)?)\s*([a-z]))\s*(?:[+\-]\s*(?:\d+(?:\.\d+)?)\s*([a-z]))?\s*(<=|>=)\s*(\d+(?:\.\d+)?)/);
    if (match) {
      let expr = line.trim().replace(new RegExp(var1, 'g'), var1).replace(new RegExp(var2, 'g'), var2);
      restricciones.push(expr);
    }
  });

  if (restricciones.length === 0) {
    throw new Error("No se detectaron restricciones válidas. Asegúrate de que tengan la forma '3x + 4y <= 100'.");
  }
  return restricciones;
}


// --- LÓGICA DE RESOLUCIÓN (Optimizada) ---
function resolver(objStr, restricciones, var1, var2, isMax) {
  const Z = (a, b) => {
    const expr = objStr.replace("Z =", "")
                        .replaceAll(var1, a)
                        .replaceAll(var2, b);
    try {
      return eval(expr);
    } catch {
      return isMax ? -Infinity : Infinity;
    }
  };

  // Estimar límites de manera más inteligente
  const limites = restricciones.map(r => {
    const match = r.match(/(\d+(?:\.\d+)?)$/);
    return match ? parseFloat(match[1]) : 0;
  });
  const maxValor = Math.max(...limites, 100) * 1.2; // Un 20% más para una mejor visualización
  const paso = Math.ceil(maxValor / 50); // Aumenta la granularidad para más precisión

  const puntos = [];
  for (let i = 0; i <= maxValor; i += paso) {
    for (let j = 0; j <= maxValor; j += paso) {
      puntos.push({ [var1]: i, [var2]: j });
    }
  }

  const puntosValidos = puntos.filter(p => {
    return restricciones.every(r => {
      const expr = r.replaceAll(var1, p[var1]).replaceAll(var2, p[var2]);
      try {
        return eval(expr);
      } catch {
        return false;
      }
    });
  });

  if (puntosValidos.length === 0) {
      throw new Error("No se encontró una región factible. Revisa las restricciones.");
  }

  const resultados = puntosValidos.map(p => {
    return { x: p[var1], y: p[var2], z: Z(p[var1], p[var2]) };
  });

  // Encuentra el óptimo de forma dinámica (max o min)
  const optimo = resultados.reduce((acc, val) => {
    if (acc === null) return val;
    return isMax ? (val.z > acc.z ? val : acc) : (val.z < acc.z ? val : acc);
  }, null);

  // Mostrar tabla y resultado final
  const outputDiv = document.getElementById("resultado");
  let tabla = `<h3>📊 Tabla de Evaluación de Puntos</h3><table><tr><th>${var1}</th><th>${var2}</th><th>Z</th></tr>`;
  resultados.forEach(r => {
    tabla += `<tr><td>${r.x.toFixed(2)}</td><td>${r.y.toFixed(2)}</td><td>${r.z.toFixed(2)}</td></tr>`;
  });
  tabla += "</table>";

  outputDiv.innerHTML = `
    <h3>✅ Resultado Óptimo</h3>
    <p>🔍 Función objetivo: <strong>${objStr}</strong></p>
    <p>➡️ Se interpretó como un problema de <strong>${isMax ? 'Maximización' : 'Minimización'}</strong>.</p>
    <p>🔝 <strong>Mejor combinación:</strong> ${var1} = ${optimo.x.toFixed(2)}, ${var2} = ${optimo.y.toFixed(2)}</p>
    <p>💡 <strong>Valor óptimo de Z:</strong> ${optimo.z.toFixed(2)}</p>
    ${tabla}
  `;

  // Dibuja la gráfica
  const ctx = document.getElementById("grafica").getContext("2d");
  const existingChart = Chart.getChart(ctx);
  if (existingChart) {
    existingChart.destroy();
  }

  new Chart(ctx, {
    type: 'scatter',
    data: {
      datasets: [
        {
          label: 'Región factible',
          data: resultados.map(r => ({ x: r.x, y: r.y })),
          backgroundColor: 'rgba(0, 123, 255, 0.5)'
        },
        {
          label: `Punto óptimo (${isMax ? 'Max' : 'Min'})`,
          data: [{ x: optimo.x, y: optimo.y }],
          backgroundColor: '#dc3545',
          pointRadius: 10,
          pointHoverRadius: 12,
        }
      ]
    },
    options: {
      scales: {
        x: {
          title: { display: true, text: var1 },
          beginAtZero: true,
          grid: { color: '#e9ecef' }
        },
        y: {
          title: { display: true, text: var2 },
          beginAtZero: true,
          grid: { color: '#e9ecef' }
        }
      },
      plugins: {
        tooltip: {
          callbacks: {
            label: function(context) {
              const point = context.raw;
              return `${context.dataset.label}: (${var1}=${point.x.toFixed(2)}, ${var2}=${point.y.toFixed(2)})`;
            }
          }
        },
        legend: {
          position: 'top',
        },
        title: {
          display: true,
          text: `Solución Gráfica del Problema`,
          font: { size: 18 }
        }
      }
    }
  });
}
</script>

</body>
</html>
