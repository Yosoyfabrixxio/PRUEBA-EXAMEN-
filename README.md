<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Método Simplex Mejorado</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
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

      const puntos = [];
      for (let i = 0; i < restricciones.length; i++) {
        for (let j = i + 1; j < restricciones.length; j++) {
          const p = interseccion(restricciones[i], restricciones[j]);
          if (p && esFactible(p, restricciones)) {
            const z = objCoefs[0]*p[0] + objCoefs[1]*p[1];
            puntos.push({ punto: p, z });
          }
        }
      }

      if (puntos.length === 0) {
        document.getElementById("resultado").innerHTML = "<b>⚠️ No hay región factible. Verifica si las restricciones se cruzan en una zona común.</b>";
        return;
      }

      puntos.sort((a, b) => tipoOpt === 'max' ? b.z - a.z : a.z - b.z);
      const optimo = puntos[0];

      // Trazar líneas de cada restricción
      const lineas = restricciones.map((r, index) => {
        const puntos = [];
        for (let x = 0; x <= Math.max(...allX) + 5; x += 0.5) {
          const a = r.coefs[0];
          const b = r.coefs[1];
          const c = r.rhs;
          if (b !== 0) {
            const y = (c - a * x) / b;
            if (y >= 0 && y <= Math.max(...allY) + 5) {
              puntos.push({ x, y });
            }
          }
        }
        return {
          label: `Restricción ${index + 1}`,
          data: puntos,
          borderColor: `hsl(${index * 60}, 70%, 50%)`,
          borderWidth: 1,
          fill: false,
          showLine: true,
          pointRadius: 0,
          type: 'line'
        };
      });

      const datasets = [
        {
          label: "Región Factible",
          data: puntos.map(p => ({ x: p.punto[0], y: p.punto[1] })),
          backgroundColor: "rgba(0, 255, 0, 0.4)",
          pointRadius: 5,
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

      const allX = puntos.map(p => p.punto[0]);
      const allY = puntos.map(p => p.punto[1]);

      chart = new Chart(ctx, {
        type: "scatter",
        data: { datasets: [...lineas, ...datasets] },
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
            legend: { display: true }
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
