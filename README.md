<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Método Simplex Mejorado</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels"></script>
  <style>
    body { font-family: Arial; padding: 20px; background: #f0f0f0; }
    input, textarea, select, button { width: 100%; padding: 10px; margin: 10px 0; }
    canvas { background: white; margin-top: 20px; }
    #resultado { background: #fff; padding: 15px; border: 1px solid #ccc; }
  </style>
</head>
<body>

  <h2>📈 Método Simplex (Con símbolos extendidos)</h2>

  <label>Tipo de Optimización:</label>
  <select id="tipoOptimizacion">
    <option value="max">Maximizar</option>
    <option value="min">Minimizar</option>
  </select>

  <label>Método de Solución:</label>
  <select id="metodoSolucion">
    <option value="grafico">Gráfico</option>
    <option value="simplex">Simplex</option>
  </select>

  <label>Función Objetivo (Ej: Z = 40x + 70y):</label>
  <input id="objetivo" placeholder="Z = 40x + 70y" value="Z = 40x + 70y">

  <label>Restricciones (una por línea):</label>
  <textarea id="restricciones" rows="6">x + 2y <= 50
4x + 3y <= 120
x >= 0
y >= 0</textarea>

  <button onclick="resolver()">Resolver y Graficar</button>

  <div id="resultado"></div>
  <canvas id="grafica" width="900" height="600"></canvas>

  <script>
    let chart;

    function parseExpresion(exp) {
      exp = exp.replace(/\s/g, '').toLowerCase();
      const coef = { x: 0, y: 0 };
      const regex = /([+-]?\d*\.?\d*)\*?([xy])/g;
      let match;
      while ((match = regex.exec(exp)) !== null) {
        let num = match[1];
        if (num === "" || num === "+") num = 1;
        else if (num === "-") num = -1;
        coef[match[2]] += parseFloat(num);
      }
      return [coef.x, coef.y];
    }

    function normalizarSimbolos(str) {
      return str.replace(/≤/g, "<=")
                .replace(/≥/g, ">=")
                .replace(/</g, "<")
                .replace(/>/g, ">")
                .replace(/=/g, "=");
    }

    function interseccion(r1, r2) {
      const [a1, b1] = r1.coefs;
      const [a2, b2] = r2.coefs;
      const c1 = r1.rhs;
      const c2 = r2.rhs;
      const det = a1 * b2 - a2 * b1;
      if (det === 0) return null;
      const x = (c1 * b2 - c2 * b1) / det;
      const y = (a1 * c2 - a2 * c1) / det;
      return [x, y];
    }

    function esFactible([x, y], restricciones) {
      for (const r of restricciones) {
        const val = r.coefs[0] * x + r.coefs[1] * y;
        if (r.tipo === "<=" && val > r.rhs + 0.001) return false;
        if (r.tipo === ">=" && val < r.rhs - 0.001) return false;
        if (r.tipo === "=" && Math.abs(val - r.rhs) > 0.001) return false;
      }
      return x >= 0 && y >= 0;
    }

    function resolver() {
      const tipoOpt = document.getElementById('tipoOptimizacion').value;
      const ctx = document.getElementById('grafica').getContext('2d');
      if (chart) chart.destroy();

      const objStr = document.getElementById("objetivo").value.trim();
      const restStr = document.getElementById("restricciones").value.trim().split("\n");

      const objCoefs = parseExpresion(objStr);

      const restricciones = [];
      for (let r of restStr) {
        r = normalizarSimbolos(r);
        const match = r.match(/(.+?)(<=|>=|=)(.+)/);
        if (!match) continue;
        const lhs = match[1].trim();
        const tipo = match[2];
        const rhs = parseFloat(match[3]);
        const coefs = parseExpresion(lhs);
        restricciones.push({ coefs, tipo, rhs });
      }

      const puntosFactibles = [];
      for (let i = 0; i < restricciones.length; i++) {
        for (let j = i + 1; j < restricciones.length; j++) {
          const p = interseccion(restricciones[i], restricciones[j]);
          if (p && esFactible(p, restricciones)) {
            const rounded = p.map(n => Math.round(n * 1000) / 1000);
            if (!puntosFactibles.some(q => q[0] === rounded[0] && q[1] === rounded[1])) {
              puntosFactibles.push(rounded);
            }
          }
        }
      }

      if (puntosFactibles.length < 3) {
        document.getElementById("resultado").innerHTML = "<b>⚠️ No hay suficientes puntos para formar una región factible.</b>";
        return;
      }

      const allX = puntosFactibles.map(p => p[0]);
      const allY = puntosFactibles.map(p => p[1]);

      const puntosZ = puntosFactibles.map(p => ({
        punto: p,
        z: objCoefs[0]*p[0] + objCoefs[1]*p[1]
      }));

      puntosZ.sort((a, b) => tipoOpt === 'max' ? b.z - a.z : a.z - b.z);
      const optimo = puntosZ[0];

      const centro = puntosFactibles.reduce((acc, p) => [acc[0]+p[0], acc[1]+p[1]], [0,0]).map(v => v / puntosFactibles.length);
      puntosFactibles.sort((a, b) => {
        const angA = Math.atan2(a[1] - centro[1], a[0] - centro[0]);
        const angB = Math.atan2(b[1] - centro[1], b[0] - centro[0]);
        return angA - angB;
      });

      const region = {
        label: "Región Factible",
        data: puntosFactibles.map(p => ({ x: p[0], y: p[1] })),
        backgroundColor: "rgba(0, 255, 0, 0.2)",
        borderColor: "green",
        fill: true,
        type: 'line',
        tension: 0.1,
        showLine: true,
        pointRadius: 0
      };

      const vertices = puntosFactibles.map((p, i) => ({
        x: p[0],
        y: p[1],
        label: String.fromCharCode(65 + i)
      }));

      const datasets = [region,
        {
          label: "Vértices",
          data: vertices,
          backgroundColor: "rgba(0, 255, 0, 0.5)",
          pointRadius: 6,
          type: "scatter"
        },
        {
          label: "Óptimo",
          data: [{ x: optimo.punto[0], y: optimo.punto[1] }],
          backgroundColor: "red",
          pointRadius: 7,
          type: "scatter"
        }
      ];

      chart = new Chart(ctx, {
        plugins: [ChartDataLabels],
        type: "scatter",
        data: { datasets },
        options: {
          responsive: true,
          scales: {
            x: {
              min: 0,
              max: Math.max(...allX) + 5,
              title: { display: true, text: 'x' }
            },
            y: {
              min: 0,
              max: Math.max(...allY) + 5,
              title: { display: true, text: 'y' }
            }
          },
          plugins: {
            datalabels: {
              display: true,
              align: 'top',
              anchor: 'center',
              font: {
                weight: 'bold',
                size: 10
              },
              formatter: function(value, context) {
                const dataset = context.dataset;
                if (dataset.label === 'Vértices') {
                  return dataset.data[context.dataIndex].label;
                }
                if (dataset.label === 'Óptimo') {
                  const point = dataset.data[context.dataIndex];
                  return `(${point.x.toFixed(1)}, ${point.y.toFixed(1)})`;
                }
                return null;
              }
            }
          }
        }
      });

      document.getElementById("resultado").innerHTML = `
        <h3>🔍 Resultado:</h3>
        <b>Tipo de optimización:</b> ${tipoOpt === 'max' ? 'Maximización' : 'Minimización'}<br>
        <b>Z ${tipoOpt === 'max' ? 'Máxima' : 'Mínima'}:</b> ${optimo.z.toFixed(2)}<br>
        <b>x:</b> ${optimo.punto[0].toFixed(2)}<br>
        <b>y:</b> ${optimo.punto[1].toFixed(2)}
      `;
    }
  </script>

</body>
</html>
