<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Método Simplex con Gráfica</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body { font-family: Arial; padding: 20px; background: #f0f0f0; }
    input, textarea, button { width: 100%; padding: 10px; margin: 10px 0; }
    canvas { background: white; margin-top: 20px; }
    #resultado { background: #fff; padding: 15px; border: 1px solid #ccc; }
  </style>
</head>
<body>

  <h2>📈 Método Simplex con Gráfica</h2>

  <label>Función Objetivo (Ej: Z = 60x + 80y):</label>
  <input id="objetivo" placeholder="Z = 60x + 80y">

  <label>Restricciones (una por línea):</label>
  <textarea id="restricciones" rows="6" placeholder="Ej:
4x + 2y <= 100
2x + 3y <= 90
x >= 0
y >= 0"></textarea>

  <button onclick="resolver()">Resolver y Graficar</button>

  <div id="resultado"></div>
  <canvas id="grafica" width="700" height="400"></canvas>

  <script>
    let chart = null;

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

    function resolver() {
      const objStr = document.getElementById("objetivo").value.trim().toLowerCase();
      const restStr = document.getElementById("restricciones").value.trim().split("\n");

      // Validación objetivo
      const objCoefs = parseExpresion(objStr);

      // Restricciones
      const restricciones = [];
      for (const r of restStr) {
        const match = r.match(/(.+)(<=|>=|=)(.+)/);
        if (!match) continue;
        const lhs = match[1].trim();
        const tipo = match[2];
        const rhs = parseFloat(match[3]);
        const coefs = parseExpresion(lhs);
        restricciones.push({ coefs, tipo, rhs });
      }

      // Calcular intersecciones factibles
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
        document.getElementById("resultado").innerHTML = "<b>No hay región factible.</b>";
        return;
      }

      puntos.sort((a, b) => b.z - a.z);
      const optimo = puntos[0];

      document.getElementById("resultado").innerHTML = `
        <h3>🔍 Resultado:</h3>
        <b>Z Máxima:</b> ${optimo.z.toFixed(2)}<br>
        <b>x:</b> ${optimo.punto[0].toFixed(2)}<br>
        <b>y:</b> ${optimo.punto[1].toFixed(2)}
      `;

      graficar(restricciones, puntos.map(p => p.punto), optimo.punto);
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

    function graficar(restricciones, puntosFactibles, puntoOptimo) {
      const datasets = [];

      for (let i = 0; i < restricciones.length; i++) {
        const r = restricciones[i];
        const x = [0, 100];
        const y = x.map(xi => {
          const a = r.coefs[0];
          const b = r.coefs[1];
          if (b === 0) return null;
          return (r.rhs - a * xi) / b;
        });

        datasets.push({
          label: `Restricción ${i + 1}`,
          data: [{x: x[0], y: y[0]}, {x: x[1], y: y[1]}],
          borderColor: 'rgba(0, 0, 255, 0.4)',
          fill: false,
          tension: 0,
        });
      }

      datasets.push({
        label: "Región Factible",
        data: puntosFactibles.map(p => ({ x: p[0], y: p[1] })),
        backgroundColor: "rgba(0, 255, 0, 0.4)",
        pointRadius: 5,
        type: "scatter"
      });

      datasets.push({
        label: "Óptimo",
        data: [{ x: puntoOptimo[0], y: puntoOptimo[1] }],
        backgroundColor: "red",
        pointRadius: 7,
        type: "scatter"
      });

      if (chart) chart.destroy();

      const ctx = document.getElementById("grafica").getContext("2d");
      chart = new Chart(ctx, {
        type: "line",
        data: { datasets },
        options: {
          responsive: true,
          scales: {
            x: { beginAtZero: true, max: 100 },
            y: { beginAtZero: true, max: 100 }
          }
        }
      });
    }
  </script>

</body>
</html>
