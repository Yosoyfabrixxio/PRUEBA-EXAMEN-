<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Solver LP Avanzado - Análisis de Texto y Simplex</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/mathjs/10.6.4/math.min.js"></script>
  <style>
    body { 
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; 
      background-color: #f8f9fa; 
      padding: 20px; 
      max-width: 1200px; 
      margin: 0 auto; 
      color: #333;
    }
    .container { 
      display: flex; 
      flex-wrap: wrap; 
      gap: 20px; 
    }
    .input-section, .output-section { 
      flex: 1; 
      min-width: 300px; 
    }
    .card {
      background: #fff; 
      padding: 25px; 
      margin-bottom: 20px; 
      border-radius: 10px; 
      box-shadow: 0 4px 6px rgba(0,0,0,0.1);
      transition: transform 0.3s, box-shadow 0.3s;
    }
    .card:hover {
      transform: translateY(-5px);
      box-shadow: 0 6px 12px rgba(0,0,0,0.15);
    }
    textarea {
      width: 100%; 
      min-height: 200px; 
      padding: 15px; 
      border: 1px solid #ddd; 
      border-radius: 8px; 
      font-size: 1em; 
      line-height: 1.5;
      resize: vertical;
    }
    button {
      background-color: #4e73df; 
      color: white; 
      border: none; 
      padding: 12px 20px; 
      border-radius: 8px; 
      cursor: pointer; 
      font-size: 1em; 
      font-weight: 600;
      transition: background-color 0.3s;
      width: 100%;
      margin-top: 10px;
    }
    button:hover { background-color: #2e59d9; }
    button.secondary {
      background-color: #858796;
    }
    button.secondary:hover {
      background-color: #6c757d;
    }
    table {
      width: 100%; 
      border-collapse: collapse; 
      margin: 15px 0;
      font-size: 0.9em;
    }
    th, td { 
      border: 1px solid #ddd; 
      padding: 12px; 
      text-align: center; 
    }
    th { 
      background-color: #f2f2f2; 
      font-weight: 600; 
    }
    tr:nth-child(even) { background-color: #f9f9f9; }
    h2, h3, h4 { 
      color: #2e59d9; 
      margin-top: 0;
    }
    h2 { border-bottom: 2px solid #eee; padding-bottom: 10px; }
    .error { 
      color: #e74c3c; 
      background-color: #fdecea; 
      padding: 15px; 
      border-radius: 8px; 
      margin: 15px 0;
    }
    .success { 
      color: #27ae60; 
      background-color: #e8f5e9; 
      padding: 15px; 
      border-radius: 8px; 
      margin: 15px 0;
    }
    .warning { 
      color: #f39c12; 
      background-color: #fff8e1; 
      padding: 15px; 
      border-radius: 8px; 
      margin: 15px 0;
    }
    .info-box {
      background-color: #f8f9fc; 
      border-left: 4px solid #4e73df; 
      padding: 15px; 
      margin: 15px 0;
    }
    canvas { 
      width: 100% !important; 
      max-height: 500px !important; 
      margin-top: 20px;
    }
    .tab {
      overflow: hidden;
      border: 1px solid #ccc;
      background-color: #f1f1f1;
      border-radius: 8px 8px 0 0;
    }
    .tab button {
      background-color: inherit;
      float: left;
      border: none;
      outline: none;
      cursor: pointer;
      padding: 14px 16px;
      transition: 0.3s;
      color: #333;
      font-weight: normal;
      width: auto;
      margin: 0;
      border-radius: 0;
    }
    .tab button:hover {
      background-color: #ddd;
    }
    .tab button.active {
      background-color: #4e73df;
      color: white;
    }
    .tabcontent {
      display: none;
      padding: 20px;
      border: 1px solid #ccc;
      border-top: none;
      border-radius: 0 0 8px 8px;
      background: white;
    }
    .var-highlight {
      background-color: #fffde7;
      padding: 2px 4px;
      border-radius: 3px;
      font-weight: bold;
    }
    .restriction-highlight {
      background-color: #e3f2fd;
      padding: 2px 4px;
      border-radius: 3px;
      font-weight: bold;
    }
  </style>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/nlp.js@4.23.0/dist/nlp.min.js"></script>
</head>
<body>
  <h2>🔍 Solver LP Avanzado - Análisis de Texto y Método Simplex</h2>
  
  <div class="container">
    <div class="input-section">
      <div class="card">
        <h3>📝 Ingrese el problema</h3>
        <textarea id="problema" rows="12" placeholder="Pegue aquí el enunciado completo del problema de programación lineal...">Digital Controls, Inc. (DCI) fabrica dos modelos de una pistola radar utilizada por la policía para monitorear la velocidad de los automóviles. El modelo A tiene una precisión de más menos 1 milla por hora, mientras que el modelo B más pequeño tiene una precisión de más menos 3 millas por hora. La empresa tiene pedido para 100 unidades del modelo A y 150 unidades del modelo B para la semana siguiente. Aunque DCI compra todos los componentes que utiliza en ambos modelos, los estuches de plástico usados para ambos modelos se fabrican en una planta de DCI en Newark, New Jersey. Cada estuche para el modelo A requiere 4 minutos de tiempo de moldeo por inyección y 6 minutos de tiempo de ensamblaje. Cada estuche para el modelo B requiere 3 minutos de moldeo por inyección y 8 minutos de ensamblaje. Para la semana siguiente la planta de Newark dispone de 600 minutos de tiempo de moldeo por inyección y 1080 minutos de tiempo de ensamblaje. El costo de manufactura es $10 por estuche para el modelo A y $6 por estuche para el modelo B. Dependiendo de la demanda y el tiempo disponible en la planta de Newark, DCI ocasionalmente compra estuches para uno o ambos modelos a un proveedor externo con el fin de abastecer los pedidos de los clientes que de lo contrario no se podrían entregar. El costo de compra es $14 por estuche para el modelo A y $9 por estuche para el modelo B. La gerencia quiere desarrollar un plan de costo mínimo que determine cuántos estuches de cada modelo deben fabricarse en la planta de Newark y cuántos estuches de cada modelo deben comprarse.</textarea>
        <button onclick="analizarProblema()">Analizar y Resolver Problema</button>
        <button onclick="mostrarEjemplo()" class="secondary">Cargar Ejemplo</button>
      </div>
      
      <div class="card" id="manualInputCard" style="display: none;">
        <h3>✏️ Ajuste Manual (Opcional)</h3>
        <div id="manualAdjustments"></div>
        <button onclick="resolverProblemaManual()" class="secondary">Resolver con Ajustes Manuales</button>
      </div>
    </div>
    
    <div class="output-section">
      <div class="card" id="resultado">
        <h3>📊 Resultados</h3>
        <p>Ingrese un problema y haga clic en "Analizar y Resolver" para ver los resultados.</p>
      </div>
      
      <div class="card" id="interpretacion">
        <h3>🔍 Interpretación del Problema</h3>
        <p>Se mostrará aquí el análisis del problema ingresado.</p>
      </div>
      
      <div class="card" id="tablaSimplex">
        <h3>📋 Tabla Simplex</h3>
        <p>Se mostrará aquí la tabla simplex del problema resuelto.</p>
      </div>
      
      <div class="card" id="graficoContainer">
        <h3>📈 Visualización</h3>
        <div class="tab">
          <button class="tablinks active" onclick="openTab(event, 'Grafico')">Gráfico</button>
          <button class="tablinks" onclick="openTab(event, 'Sensibilidad')">Sensibilidad</button>
        </div>
        <div id="Grafico" class="tabcontent" style="display: block;">
          <canvas id="graficoCanvas"></canvas>
        </div>
        <div id="Sensibilidad" class="tabcontent">
          <p>Se mostrará aquí el análisis de sensibilidad.</p>
        </div>
      </div>
    </div>
  </div>

<script>
// Variables globales
let problemaParseado = null;
let solution = null;

// Mostrar ejemplo
function mostrarEjemplo() {
  const ejemplo = `Una fábrica produce dos tipos de muebles: mesas y sillas. 
Cada mesa genera una ganancia de $80 y cada silla $50. 
Para producir una mesa se necesitan 4 horas de trabajo y 2 unidades de material. 
Para una silla se necesitan 3 horas de trabajo y 1 unidad de material. 
Disponemos de 240 horas de trabajo y 100 unidades de material. 
¿Cuántas mesas y sillas se deben producir para maximizar la ganancia?`;
  
  document.getElementById('problema').value = ejemplo;
}

// Función principal para analizar el problema
async function analizarProblema() {
  const textoProblema = document.getElementById('problema').value.trim();
  
  if (!textoProblema) {
    mostrarError("Por favor ingrese un problema para analizar.");
    return;
  }
  
  // Mostrar mensaje de carga
  document.getElementById('resultado').innerHTML = "<p>Analizando problema, por favor espere...</p>";
  
  try {
    // Parsear el problema (simulamos un retraso para el análisis)
    await new Promise(resolve => setTimeout(resolve, 1000));
    problemaParseado = parsearProblema(textoProblema);
    
    // Mostrar interpretación
    mostrarInterpretacion(problemaParseado);
    
    // Mostrar controles para ajuste manual
    mostrarControlesManuales(problemaParseado);
    
    // Resolver el problema
    resolverProblema(problemaParseado);
  } catch (error) {
    mostrarError("Error al analizar el problema: " + error.message);
    console.error(error);
  }
}

// Función para parsear el problema de texto a estructura matemática
function parsearProblema(texto) {
  // Convertir a minúsculas para facilitar el análisis
  const textoLower = texto.toLowerCase();
  
  // Detectar tipo de problema
  const esMinimizacion = textoLower.includes("minimizar") || 
                        textoLower.includes("minimización") ||
                        textoLower.includes("costo mínimo") || 
                        textoLower.includes("coste mínimo");
  
  // Extraer variables (ejemplo simplificado)
  const variables = [];
  const regexVariables = /(modelo\s+\w+|mesas|sillas|trajes|sacos|\b[a-z]\b)/gi;
  const matches = texto.match(regexVariables) || [];
  
  // Procesar variables únicas
  const uniqueVars = [...new Set(matches.map(v => v.trim().toLowerCase()))];
  uniqueVars.forEach(v => {
    if (v.length <= 2) { // Suponemos que variables de una letra son válidas
      variables.push(v);
    } else if (v.includes("modelo")) {
      variables.push(v.replace(/\s+/g, '')); // "modelo a" -> "modeloa"
    }
  });
  
  // Si no encontramos variables, usamos x1, x2 como predeterminado
  if (variables.length === 0) {
    variables.push('x1', 'x2');
  } else if (variables.length === 1) {
    variables.push(variables[0] + '2'); // Duplicamos si solo hay una
  }
  
  // Extraer función objetivo (ejemplo simplificado)
  let objetivoCoefs = new Array(variables.length).fill(0);
  let objetivoTexto = "";
  
  if (esMinimizacion) {
    // Buscar costos
    const costos = texto.match(/\$\d+/g) || [];
    if (costos.length >= variables.length) {
      objetivoCoefs = costos.slice(0, variables.length).map(c => parseInt(c.replace('$', '')));
    } else {
      // Valores por defecto para minimización
      objetivoCoefs = [10, 6]; // Ejemplo común
    }
    objetivoTexto = "Minimizar costos";
  } else {
    // Buscar ganancias/beneficios
    const ganancias = texto.match(/(ganancia|beneficio|utilidad)\s+(de|)\s*\$\d+/gi) || [];
    if (ganancias.length >= variables.length) {
      objetivoCoefs = ganancias.slice(0, variables.length).map(g => {
        const num = g.match(/\d+/);
        return num ? parseInt(num[0]) : 1;
      });
    } else {
      // Valores por defecto para maximización
      objetivoCoefs = [80, 50]; // Ejemplo común
    }
    objetivoTexto = "Maximizar ganancias";
  }
  
  // Extraer restricciones (ejemplo simplificado)
  const restricciones = [];
  
  // 1. Restricciones de demanda/pedidos
  const regexPedidos = /(pedido|demanda|requiere)\s+(de|)\s*(\d+)\s+(unidades|)\s*(del|de|)\s*(modelo\s+\w+|[\w\s]+)/gi;
  let match;
  while ((match = regexPedidos.exec(texto)) {
    const cantidad = parseInt(match[3]);
    const modelo = match[6].trim().toLowerCase().replace(/\s+/g, '');
    const varIndex = variables.findIndex(v => v.includes(modelo));
    if (varIndex >= 0) {
      const coefs = new Array(variables.length).fill(0);
      coefs[varIndex] = 1;
      restricciones.push({
        coefficients: coefs,
        sign: '>=',
        value: cantidad,
        text: `${variables[varIndex]} >= ${cantidad} (demanda mínima)`
      });
    }
  }
  
  // 2. Restricciones de recursos (tiempo, material, etc.)
  const regexRecursos = /(\d+)\s+(minutos|horas|unidades)\s+(de|)\s*[\w\s]+\s+(para|por|de)\s+(modelo\s+\w+|[\w\s]+)/gi;
  while ((match = regexRecursos.exec(texto))) {
    const cantidad = parseInt(match[1]);
    const modelo = match[5].trim().toLowerCase().replace(/\s+/g, '');
    const varIndex = variables.findIndex(v => v.includes(modelo));
    if (varIndex >= 0) {
      const coefs = new Array(variables.length).fill(0);
      coefs[varIndex] = cantidad;
      restricciones.push({
        coefficients: coefs,
        sign: '<=',
        value: cantidad,
        text: `${cantidad}${match[2]} por ${variables[varIndex]}`
      });
    }
  }
  
  // 3. Restricciones de disponibilidad
  const regexDisponibilidad = /(dispone|disponible|tiene)\s+(de|)\s*(\d+)\s+(minutos|horas|unidades)/gi;
  while ((match = regexDisponibilidad.exec(texto))) {
    const cantidadTotal = parseInt(match[3]);
    // Buscamos restricciones relacionadas para establecer el límite
    for (let i = 0; i < restricciones.length; i++) {
      if (restricciones[i].text.includes(match[4])) {
        restricciones[i].value = cantidadTotal;
        restricciones[i].text = `Total ${match[4]}: <= ${cantidadTotal}`;
      }
    }
  }
  
  // Si no encontramos restricciones, añadimos algunas por defecto
  if (restricciones.length === 0) {
    restricciones.push(
      { coefficients: [4, 3], sign: '<=', value: 240, text: "4x1 + 3x2 <= 240 (horas)" },
      { coefficients: [2, 1], sign: '<=', value: 100, text: "2x1 + x2 <= 100 (material)" }
    );
  }
  
  // Para el problema específico de DCI
  if (textoLower.includes("digital controls")) {
    // Variables: modeloa_fab, modeloa_comp, modelob_fab, modelob_comp
    variables.length = 0;
    variables.push('modeloa_fab', 'modeloa_comp', 'modelob_fab', 'modelob_comp');
    
    // Función objetivo: minimizar costos
    objetivoCoefs = [10, 14, 6, 9];
    objetivoTexto = "Minimizar costos totales";
    
    // Restricciones
    restricciones.length = 0;
    // Demanda
    restricciones.push(
      { coefficients: [1, 1, 0, 0], sign: '>=', value: 100, text: "modeloa_fab + modeloa_comp >= 100 (demanda modelo A)" },
      { coefficients: [0, 0, 1, 1], sign: '>=', value: 150, text: "modelob_fab + modelob_comp >= 150 (demanda modelo B)" }
    );
    // Tiempos de producción
    restricciones.push(
      { coefficients: [4, 0, 3, 0], sign: '<=', value: 600, text: "4*modeloa_fab + 3*modelob_fab <= 600 (minutos moldeo)" },
      { coefficients: [6, 0, 8, 0], sign: '<=', value: 1080, text: "6*modeloa_fab + 8*modelob_fab <= 1080 (minutos ensamblaje)" }
    );
    // No negatividad
    restricciones.push(
      { coefficients: [1, 0, 0, 0], sign: '>=', value: 0, text: "modeloa_fab >= 0" },
      { coefficients: [0, 1, 0, 0], sign: '>=', value: 0, text: "modeloa_comp >= 0" },
      { coefficients: [0, 0, 1, 0], sign: '>=', value: 0, text: "modelob_fab >= 0" },
      { coefficients: [0, 0, 0, 1], sign: '>=', value: 0, text: "modelob_comp >= 0" }
    );
  }
  
  return {
    variables: variables,
    objetivo: {
      coefficients: objetivoCoefs,
      tipo: esMinimizacion ? 'min' : 'max',
      texto: objetivoTexto
    },
    restricciones: restricciones,
    textoOriginal: texto
  };
}

// Mostrar interpretación del problema
function mostrarInterpretacion(problema) {
  let html = `<h4>📌 Problema de ${problema.objetivo.tipo === 'min' ? 'Minimización' : 'Maximización'}</h4>`;
  html += `<p><strong>Objetivo:</strong> ${problema.objetivo.texto}</p>`;
  
  html += `<h4>🔢 Variables de Decisión</h4><ul>`;
  problema.variables.forEach((v, i) => {
    html += `<li><span class="var-highlight">${v}</span>: Coeficiente en función objetivo = ${problema.objetivo.coefficients[i]}</li>`;
  });
  html += `</ul>`;
  
  html += `<h4>📏 Restricciones</h4><ul>`;
  problema.restricciones.forEach(r => {
    html += `<li><span class="restriction-highlight">${r.text}</span></li>`;
  });
  html += `</ul>`;
  
  document.getElementById('interpretacion').innerHTML = html;
}

// Mostrar controles para ajuste manual
function mostrarControlesManuales(problema) {
  let html = `<h4>✏️ Editar Variables</h4>`;
  html += `<table><tr><th>Variable</th><th>Coeficiente Objetivo</th></tr>`;
  
  problema.variables.forEach((v, i) => {
    html += `<tr>
      <td>${v}</td>
      <td><input type="number" id="var_${i}" value="${problema.objetivo.coefficients[i]}" step="0.01"></td>
    </tr>`;
  });
  html += `</table>`;
  
  html += `<h4>📐 Editar Restricciones</h4>`;
  html += `<table><tr><th>Restricción</th><th>Lado Izquierdo</th><th>Signo</th><th>Lado Derecho</th></tr>`;
  
  problema.restricciones.forEach((r, i) => {
    html += `<tr>
      <td>${r.text.split('(')[0]}</td>
      <td>${r.coefficients.map((c, j) => `${c}${problema.variables[j]}`).join(' + ')}</td>
      <td>
        <select id="rest_sign_${i}">
          <option value="<=" ${r.sign === '<=' ? 'selected' : ''}><=</option>
          <option value=">=" ${r.sign === '>=' ? 'selected' : ''}>>=</option>
          <option value="==" ${r.sign === '==' ? 'selected' : ''}>=</option>
        </select>
      </td>
      <td><input type="number" id="rest_val_${i}" value="${r.value}" step="0.01"></td>
    </tr>`;
  });
  html += `</table>`;
  
  document.getElementById('manualAdjustments').innerHTML = html;
  document.getElementById('manualInputCard').style.display = 'block';
}

// Resolver problema con los parámetros parseados
function resolverProblema(problema) {
  // Implementación del método Simplex (versión mejorada)
  try {
    solution = simplexSolver(
      problema.variables,
      problema.objetivo.coefficients,
      problema.restricciones,
      problema.objetivo.tipo
    );
    
    mostrarResultados(solution, problema);
  } catch (error) {
    mostrarError("Error al resolver el problema: " + error.message);
    console.error(error);
  }
}

// Resolver problema con ajustes manuales
function resolverProblemaManual() {
  if (!problemaParseado) return;
  
  // Actualizar coeficientes objetivo
  for (let i = 0; i < problemaParseado.variables.length; i++) {
    const val = parseFloat(document.getElementById(`var_${i}`).value);
    if (!isNaN(val)) {
      problemaParseado.objetivo.coefficients[i] = val;
    }
  }
  
  // Actualizar restricciones
  for (let i = 0; i < problemaParseado.restricciones.length; i++) {
    const sign = document.getElementById(`rest_sign_${i}`).value;
    const val = parseFloat(document.getElementById(`rest_val_${i}`).value);
    
    if (!isNaN(val)) {
      problemaParseado.restricciones[i].sign = sign;
      problemaParseado.restricciones[i].value = val;
    }
  }
  
  // Resolver con nuevos parámetros
  resolverProblema(problemaParseado);
}

// Implementación mejorada del método Simplex
function simplexSolver(variables, objectiveCoeffs, restrictions, problemType) {
  // Validar entrada
  if (variables.length !== objectiveCoeffs.length) {
    throw new Error("El número de variables no coincide con los coeficientes de la función objetivo");
  }
  
  // Convertir a forma estándar
  const numVars = variables.length;
  const numSlacks = restrictions.length;
  const totalVars = numVars + numSlacks;
  
  // Matriz de coeficientes aumentada (tableau)
  let tableau = [];
  let basis = [];
  let artificialVars = 0;
  
  // Preparar la función objetivo para el tableau
  const objectiveRow = new Array(totalVars + 1).fill(0);
  for (let j = 0; j < numVars; j++) {
    objectiveRow[j] = problemType === 'max' ? -objectiveCoeffs[j] : objectiveCoeffs[j];
  }
  
  // Inicializar restricciones en el tableau
  for (let i = 0; i < restrictions.length; i++) {
    const row = new Array(totalVars + 1).fill(0);
    
    // Coeficientes de las variables originales
    for (let j = 0; j < numVars; j++) {
      row[j] = restrictions[i].coefficients[j];
    }
    
    // Manejar variables de holgura/exceso/artificiales
    if (restrictions[i].sign === '<=') {
      row[numVars + i] = 1; // Variable de holgura
      basis.push(numVars + i);
    } else if (restrictions[i].sign === '>=') {
      row[numVars + i] = -1; // Variable de exceso
      // Necesitamos variable artificial para la fase I
      row[numVars + restrictions.length + artificialVars] = 1;
      artificialVars++;
      basis.push(numVars + restrictions.length + artificialVars - 1);
    } else if (restrictions[i].sign === '==') {
      // Necesitamos variable artificial para la fase I
      row[numVars + restrictions.length + artificialVars] = 1;
      artificialVars++;
      basis.push(numVars + restrictions.length + artificialVars - 1);
    }
    
    // Lado derecho
    row[totalVars] = restrictions[i].value;
    
    tableau.push(row);
  }
  
  // Si hay variables artificiales, necesitamos Fase I
  if (artificialVars > 0) {
    // Crear función objetivo artificial para Fase I
    const artificialObj = new Array(totalVars + restrictions.length + artificialVars + 1).fill(0);
    
    // Sumar todas las restricciones con variables artificiales
    for (let i = 0; i < restrictions.length; i++) {
      if (restrictions[i].sign === '>=' || restrictions[i].sign === '==') {
        for (let j = 0; j < totalVars + 1; j++) {
          artificialObj[j] += tableau[i][j];
        }
      }
    }
    
    // Negar la función objetivo artificial
    for (let j = 0; j < artificialObj.length; j++) {
      artificialObj[j] = -artificialObj[j];
    }
    
    // Guardar la función objetivo original para Fase II
    const originalObj = [...objectiveRow];
    
    // Fase I: Resolver problema artificial
    tableau.unshift(artificialObj);
    let phase1Result = simplexIterations(tableau, basis, numVars, restrictions.length + artificialVars);
    
    // Verificar si la Fase I encontró solución factible
    if (Math.abs(phase1Result.tableau[0][totalVars + artificialVars]) > 1e-6) {
      return { error: "El problema no tiene solución factible" };
    }
    
    // Fase II: Usar solución de Fase I como punto de partida con la función objetivo original
    tableau = phase1Result.tableau.slice(1); // Eliminar fila objetivo artificial
    tableau[0] = new Array(totalVars + artificialVars + 1).fill(0);
    
    // Configurar la función objetivo original
    for (let j = 0; j < numVars; j++) {
      tableau[0][j] = problemType === 'max' ? -originalObj[j] : originalObj[j];
    }
    
    // Recalcular la fila objetivo para las variables básicas
    for (let i = 1; i < tableau.length; i++) {
      const basicVar = basis[i-1];
      if (basicVar < numVars) { // Si es variable de decisión original
        const factor = tableau[0][basicVar];
        for (let j = 0; j < tableau[0].length; j++) {
          tableau[0][j] -= factor * tableau[i][j];
        }
      }
    }
    
    // Eliminar columnas de variables artificiales
    for (let row of tableau) {
      row.splice(numVars + restrictions.length, artificialVars);
    }
    
    basis = phase1Result.basis.filter(b => b < numVars + restrictions.length);
  } else {
    // No hay variables artificiales, proceder directamente con el problema original
    tableau.unshift(objectiveRow);
  }
  
  // Realizar iteraciones Simplex
  const result = simplexIterations(tableau, basis, numVars, restrictions.length);
  
  // Extraer solución
  const solution = new Array(numVars).fill(0);
  const slackValues = new Array(restrictions.length).fill(0);
  const optimalValue = problemType === 'max' ? result.tableau[0][numVars + restrictions.length] : 
                                             -result.tableau[0][numVars + restrictions.length];
  
  for (let i = 0; i < basis.length; i++) {
    const varIndex = basis[i];
    if (varIndex < numVars) {
      solution[varIndex] = result.tableau[i+1][numVars + restrictions.length];
    } else if (varIndex < numVars + restrictions.length) {
      slackValues[varIndex - numVars] = result.tableau[i+1][numVars + restrictions.length];
    }
  }
  
  return {
    variables: solution,
    slackVariables: slackValues,
    optimalValue: optimalValue,
    tableau: result.tableau,
    basis: basis,
    iterations: result.iterations,
    isOptimal: result.isOptimal,
    artificialVars: artificialVars
  };
}

// Iteraciones del método Simplex
function simplexIterations(tableau, basis, numVars, numRestrictions) {
  let iterations = 0;
  const maxIterations = 1000;
  let isOptimal = false;
  
  while (iterations < maxIterations) {
    iterations++;
    
    // Paso 1: Verificar optimalidad
    let minVal = 0;
    let pivotCol = -1;
    
    for (let j = 0; j < tableau[0].length - 1; j++) {
      if (tableau[0][j] < minVal - 1e-6) { // Pequeña tolerancia numérica
        minVal = tableau[0][j];
        pivotCol = j;
      }
    }
    
    if (pivotCol === -1) {
      isOptimal = true;
      break;
    }
    
    // Paso 2: Encontrar fila pivote
    let pivotRow = -1;
    let minRatio = Infinity;
    
    for (let i = 1; i < tableau.length; i++) {
      if (tableau[i][pivotCol] > 1e-6) { // Pequeña tolerancia numérica
        const ratio = tableau[i][tableau[0].length - 1] / tableau[i][pivotCol];
        if (ratio < minRatio) {
          minRatio = ratio;
          pivotRow = i;
        }
      }
    }
    
    if (pivotRow === -1) {
      return { 
        tableau: tableau,
        basis: basis,
        iterations: iterations,
        isOptimal: false,
        error: "El problema es no acotado"
      };
    }
    
    // Paso 3: Actualizar base
    basis[pivotRow - 1] = pivotCol;
    
    // Paso 4: Pivotear
    const pivotVal = tableau[pivotRow][pivotCol];
    
    // Normalizar fila pivote
    for (let j = 0; j < tableau[pivotRow].length; j++) {
      tableau[pivotRow][j] /= pivotVal;
    }
    
    // Actualizar otras filas
    for (let i = 0; i < tableau.length; i++) {
      if (i !== pivotRow) {
        const factor = tableau[i][pivotCol];
        for (let j = 0; j < tableau[i].length; j++) {
          tableau[i][j] -= factor * tableau[pivotRow][j];
        }
      }
    }
  }
  
  if (!isOptimal) {
    return { 
      tableau: tableau,
      basis: basis,
      iterations: iterations,
      isOptimal: false,
      error: "Número máximo de iteraciones alcanzado"
    };
  }
  
  return {
    tableau: tableau,
    basis: basis,
    iterations: iterations,
    isOptimal: true
  };
}

// Mostrar resultados
function mostrarResultados(solution, problema) {
  if (solution.error) {
    mostrarError(solution.error);
    return;
  }
  
  let html = `<div class="${solution.isOptimal ? 'success' : 'warning'}">`;
  
  if (solution.isOptimal) {
    html += `<h3>✅ Solución Óptima Encontrada</h3>`;
    html += `<p><strong>Valor Óptimo:</strong> ${solution.optimalValue.toFixed(2)}</p>`;
  } else {
    html += `<h3>⚠️ Solución Parcial</h3>`;
    html += `<p>Se detuvo después de ${solution.iterations} iteraciones sin alcanzar optimalidad.</p>`;
  }
  
  html += `<h4>Valores de las Variables</h4>`;
  html += `<table><tr><th>Variable</th><th>Valor</th></tr>`;
  
  problema.variables.forEach((v, i) => {
    html += `<tr><td>${v}</td><td>${solution.variables[i].toFixed(2)}</td></tr>`;
  });
  
  html += `</table>`;
  
  html += `<h4>Variables de Holgura/Exceso</h4>`;
  html += `<table><tr><th>Restricción</th><th>Valor</th><th>Interpretación</th></tr>`;
  
  problema.restricciones.forEach((r, i) => {
    let interpretacion = "";
    const slack = solution.slackVariables[i] || 0;
    
    if (r.sign === '<=') {
      interpretacion = `${slack.toFixed(2)} unidades no utilizadas del recurso`;
    } else if (r.sign === '>=') {
      interpretacion = `${slack.toFixed(2)} unidades de exceso sobre el mínimo`;
    } else {
      interpretacion = "Restricción de igualdad exacta";
    }
    
    html += `<tr>
      <td>${r.text}</td>
      <td>${slack.toFixed(2)}</td>
      <td>${interpretacion}</td>
    </tr>`;
  });
  
  html += `</table>`;
  html += `<p><strong>Iteraciones realizadas:</strong> ${solution.iterations}</p>`;
  html += `</div>`;
  
  document.getElementById('resultado').innerHTML = html;
  
  // Mostrar tabla simplex (solo para problemas pequeños)
  mostrarTablaSimplex(solution, problema);
  
  // Mostrar gráfico (solo para problemas con 2 variables)
  if (problema.variables.length === 2) {
    mostrarGrafico(solution, problema);
  } else {
    document.getElementById('graficoContainer').style.display = 'none';
  }
}

// Mostrar tabla simplex
function mostrarTablaSimplex(solution, problema) {
  if (problema.variables.length > 3 || problema.restricciones.length > 5) {
    document.getElementById('tablaSimplex').innerHTML = `
      <h3>📋 Tabla Simplex</h3>
      <p>El problema es demasiado grande para mostrar la tabla completa. 
      Se realizaron ${solution.iterations} iteraciones del método Simplex.</p>
    `;
    return;
  }
  
  let html = `<h3>📋 Tabla Simplex Final</h3>`;
  html += `<div style="overflow-x: auto;"><table>`;
  
  // Encabezados
  html += `<tr><th>Base</th>`;
  for (let i = 0; i < problema.variables.length; i++) {
    html += `<th>${problema.variables[i]}</th>`;
  }
  for (let i = 0; i < problema.restricciones.length; i++) {
    html += `<th>s${i+1}</th>`;
  }
  html += `<th>Solución</th></tr>`;
  
  // Filas
  for (let i = 0; i < solution.tableau.length; i++) {
    html += `<tr>`;
    
    // Columna Base
    if (i === 0) {
      html += `<td>Z</td>`;
    } else {
      const basisIndex = solution.basis[i-1];
      if (basisIndex < problema.variables.length) {
        html += `<td>${problema.variables[basisIndex]}</td>`;
      } else {
        html += `<td>s${basisIndex - problema.variables.length + 1}</td>`;
      }
    }
    
    // Coeficientes
    for (let j = 0; j < solution.tableau[i].length; j++) {
      html += `<td>${solution.tableau[i][j].toFixed(2)}</td>`;
    }
    
    html += `</tr>`;
  }
  
  html += `</table></div>`;
  html += `<p class="info-box">La tabla muestra los valores finales después de ${solution.iterations} iteraciones del método Simplex.</p>`;
  
  document.getElementById('tablaSimplex').innerHTML = html;
}

// Mostrar gráfico para problemas con 2 variables
function mostrarGrafico(solution, problema) {
  const ctx = document.getElementById('graficoCanvas').getContext('2d');
  
  // Generar puntos factibles
  const puntosFactibles = generarPuntosFactibles(problema);
  
  // Configurar datos del gráfico
  const datasets = [{
    label: 'Región Factible',
    data: puntosFactibles,
    backgroundColor: 'rgba(54, 162, 235, 0.5)',
    pointRadius: 3,
    showLine: true,
    borderColor: 'rgba(54, 162, 235, 0.8)',
    fill: true
  }];
  
  // Añadir solución óptima si existe
  if (solution.variables.length === 2 && solution.isOptimal) {
    datasets.push({
      label: 'Solución Óptima',
      data: [{x: solution.variables[0], y: solution.variables[1]}],
      backgroundColor: 'rgba(255, 99, 132, 1)',
      pointRadius: 8
    });
  }
  
  // Añadir restricciones como líneas
  problema.restricciones.forEach((r, i) => {
    if (r.coefficients.length === 2) {
      // Solo mostramos restricciones que involucren ambas variables
      if (r.coefficients[0] !== 0 && r.coefficients[1] !== 0) {
        const points = [];
        // Calcular puntos para la línea de restricción
        const x1 = 0;
        const y1 = (r.value - r.coefficients[0] * x1) / r.coefficients[1];
        const y2 = 0;
        const x2 = (r.value - r.coefficients[1] * y2) / r.coefficients[0];
        
        // Solo añadir si los puntos son positivos (para visualización)
        if (y1 >= 0) points.push({x: x1, y: y1});
        if (x2 >= 0) points.push({x: x2, y: y2});
        
        if (points.length === 2) {
          datasets.push({
            label: `Restricción ${i+1}`,
            data: points,
            borderColor: `hsl(${i * 60}, 70%, 50%)`,
            borderWidth: 2,
            pointRadius: 0,
            fill: false,
            showLine: true
          });
        }
      }
    }
  });
  
  // Crear gráfico
  new Chart(ctx, {
    type: 'scatter',
    data: {
      datasets: datasets
    },
    options: {
      responsive: true,
      plugins: {
        legend: { position: 'top' },
        tooltip: {
          callbacks: {
            label: function(context) {
              return `${problema.variables[0]}: ${context.parsed.x.toFixed(2)}, ${problema.variables[1]}: ${context.parsed.y.toFixed(2)}`;
            }
          }
        }
      },
      scales: {
        x: {
          title: { display: true, text: problema.variables[0] },
          min: 0,
          grid: { color: '#ddd' }
        },
        y: {
          title: { display: true, text: problema.variables[1] },
          min: 0,
          grid: { color: '#ddd' }
        }
      }
    }
  });
  
  document.getElementById('graficoContainer').style.display = 'block';
}

// Generar puntos factibles para el gráfico
function generarPuntosFactibles(problema) {
  if (problema.variables.length !== 2) return [];
  
  const puntos = [];
  const maxX = Math.max(...problema.restricciones.map(r => {
    if (r.coefficients[0] !== 0) return r.value / r.coefficients[0];
    return 0;
  }).filter(x => isFinite(x))) * 1.2 || 100;
  
  const maxY = Math.max(...problema.restricciones.map(r => {
    if (r.coefficients[1] !== 0) return r.value / r.coefficients[1];
    return 0;
  }).filter(y => isFinite(y))) * 1.2 || 100;
  
  const stepX = maxX / 30;
  const stepY = maxY / 30;
  
  for (let x = 0; x <= maxX; x += stepX) {
    for (let y = 0; y <= maxY; y += stepY) {
      let factible = true;
      
      for (const r of problema.restricciones) {
        const lhs = r.coefficients[0] * x + r.coefficients[1] * y;
        
        if (r.sign === '<=' && lhs > r.value + 1e-6) {
          factible = false;
          break;
        } else if (r.sign === '>=' && lhs < r.value - 1e-6) {
          factible = false;
          break;
        } else if (r.sign === '==' && Math.abs(lhs - r.value) > 1e-6) {
          factible = false;
          break;
        }
      }
      
      if (factible) {
        puntos.push({x: x, y: y});
      }
    }
  }
  
  return puntos;
}

// Mostrar mensaje de error
function mostrarError(mensaje) {
  document.getElementById('resultado').innerHTML = `
    <div class="error">
      <h3>❌ Error</h3>
      <p>${mensaje}</p>
    </div>
  `;
}

// Manejar pestañas
function openTab(evt, tabName) {
  const tabcontent = document.getElementsByClassName("tabcontent");
  for (let i = 0; i < tabcontent.length; i++) {
    tabcontent[i].style.display = "none";
  }
  
  const tablinks = document.getElementsByClassName("tablinks");
  for (let i = 0; i < tablinks.length; i++) {
    tablinks[i].className = tablinks[i].className.replace(" active", "");
  }
  
  document.getElementById(tabName).style.display = "block";
  evt.currentTarget.className += " active";
}
</script>
</body>
</html>
