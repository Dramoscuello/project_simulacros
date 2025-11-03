# Generador de Banco de Preguntas ICFES - Matemáticas

## Contexto del Proyecto

En Colombia se realizan anualmente las **Pruebas de Estado ICFES (Saber 11°)** para estudiantes de grado 11, evaluando competencias en las áreas de Lectura Crítica, Matemáticas, Sociales y Ciudadanas, Ciencias Naturales e Inglés.

Te adjunto:
- 📘 Guía oficial del Ministerio de Educación Nacional
- Los niveles de desempeño en matemáticas
- Marco de referencia
- 📄 Pruebas de matemáticas de años anteriores (PDF)

## Objetivo Principal

Analizar los materiales adjuntos y **generar un banco de preguntas originales** para el área de **Matemáticas**, siguiendo los patrones, marco de referencia y estándares del ICFES, pero con innovación en los contextos, datos y situaciones planteadas.

**⚠️ IMPORTANTE**: No se trata de replicar preguntas existentes, sino de **innovar manteniendo la estructura, competencias, niveles de desempeño y nivel de dificultad** observados en las pruebas oficiales.

---

## Competencias Matemáticas a Evaluar

Según el marco de referencia del ICFES, las preguntas deben evaluar:

1. **Comunicación, representación y modelación**
2. **Planteamiento y resolución de problemas**
3. **Razonamiento y argumentación**

Aplicadas en los siguientes **componentes**:
- Numérico-variacional
- Geométrico-métrico
- Aleatorio

---

## Estructura de Base de Datos (Supabase)

### Tabla `estudiantes`
```sql
CREATE TABLE estudiantes (
  documento INT8 PRIMARY KEY,
  nombre VARCHAR NOT NULL
);
```

**Ejemplo de datos:**
```
| documento  | nombre              |
|------------|---------------------|
| 1234567890 | Juan Pérez García   |
| 9876543210 | María López Ruiz    |
```

### Tabla `respuestas_estudiantes`
```sql
CREATE TABLE respuestas_estudiantes (
  id SERIAL PRIMARY KEY,
  documento_estudiante INT8 REFERENCES estudiantes(documento),
  respuestas JSONB NOT NULL
);
```

**Ejemplo de datos después del envío:**
```json
{
  "id": 1,
  "documento_estudiante": 1234567890,
  "respuestas": {
    "1": "B",
    "2": "A",
    "3": "C",
    "4": "D",
    "5": "A",
    ...
    "30": "B"
  }
}
```

---

## Especificaciones Técnicas

### 1. Estructura de las Preguntas

Cada pregunta debe incluir:
```json
{
  "numero": 1,
  "competencia": "Razonamiento cuantitativo",
  "componente": "Numérico-variacional",
  "tema": "Álgebra - Sistemas de ecuaciones",
  "dificultad": "media",
  "contexto": "Situación contextualizada (2-4 líneas)",
  "tiene_grafico": true,
  "tipo_grafico": "tabla|barras|circular|lineas|geometrico|plano_cartesiano",
  "pregunta": "Texto de la pregunta",
  "opciones": {
    "A": "Opción A",
    "B": "Opción B",
    "C": "Opción C",
    "D": "Opción D"
  },
  "respuesta_correcta": "B",
  "justificacion": "Explicación breve de por qué B es correcta"
}
```

### 2. Generación de Gráficos

**CRÍTICO**: Aproximadamente el **30-40% de las preguntas** deben incluir elementos visuales (tablas, gráficos, diagramas).

Debes generar los gráficos usando **código embebido** en el artefacto HTML:

#### Opciones tecnológicas:
- **Chart.js** → Para gráficos estadísticos (barras, líneas, circulares)
- **SVG** → Para figuras geométricas, diagramas, planos cartesianos
- **Canvas** → Para representaciones matemáticas complejas
- **Tablas HTML + CSS** → Para datos tabulados

#### Ejemplo de implementación:
```javascript
// Gráfico de barras con Chart.js
const ctx = document.getElementById('grafico-pregunta-5').getContext('2d');
new Chart(ctx, {
  type: 'bar',
  data: {
    labels: ['Producto A', 'Producto B', 'Producto C'],
    datasets: [{
      label: 'Ventas (millones)',
      data: [12, 19, 8],
      backgroundColor: ['#FF6384', '#36A2EB', '#FFCE56']
    }]
  },
  options: {
    responsive: true,
    maintainAspectRatio: false,
    scales: {
      y: { beginAtZero: true }
    }
  }
});
```
```html
<!-- Figura geométrica con SVG -->
<svg width="300" height="200" class="grafico-geometrico">
  <circle cx="150" cy="100" r="80" fill="none" stroke="#2196F3" stroke-width="2"/>
  <line x1="150" y1="100" x2="230" y2="100" stroke="#FF5722" stroke-width="2"/>
  <text x="190" y="95" font-size="14">r = 5 cm</text>
</svg>
```

---

## Artefacto HTML a Generar

Crea un **artefacto HTML completo** con las siguientes características:

### Estructura del Artefacto
```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Simulacro ICFES - Matemáticas</title>
  
  <!-- Chart.js para gráficos -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
  
  <!-- Supabase JS Client -->
  <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
  
  <style>
    /* ============================================ */
    /* ESTILOS PROFESIONALES TIPO ICFES */
    /* ============================================ */
    
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }
    
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      min-height: 100vh;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    
    /* ============================================ */
    /* MODAL OVERLAY */
    /* ============================================ */
    
    .modal-overlay {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(0, 0, 0, 0.8);
      display: flex;
      justify-content: center;
      align-items: center;
      z-index: 1000;
      animation: fadeIn 0.3s ease;
    }
    
    @keyframes fadeIn {
      from { opacity: 0; }
      to { opacity: 1; }
    }
    
    .modal-contenido {
      background: white;
      padding: 2.5rem;
      border-radius: 12px;
      max-width: 450px;
      width: 90%;
      box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
      animation: slideUp 0.4s ease;
    }
    
    @keyframes slideUp {
      from { transform: translateY(50px); opacity: 0; }
      to { transform: translateY(0); opacity: 1; }
    }
    
    .modal-contenido h2 {
      color: #333;
      margin-bottom: 1rem;
      font-size: 1.5rem;
    }
    
    .modal-contenido p {
      color: #666;
      margin-bottom: 1.5rem;
      line-height: 1.6;
    }
    
    /* ============================================ */
    /* FORMULARIOS */
    /* ============================================ */
    
    .form-group {
      margin-bottom: 1.5rem;
    }
    
    .form-group label {
      display: block;
      margin-bottom: 0.5rem;
      color: #333;
      font-weight: 600;
    }
    
    .form-group input {
      width: 100%;
      padding: 0.75rem;
      border: 2px solid #e0e0e0;
      border-radius: 8px;
      font-size: 1rem;
      transition: border-color 0.3s;
    }
    
    .form-group input:focus {
      outline: none;
      border-color: #667eea;
    }
    
    /* ============================================ */
    /* BOTONES */
    /* ============================================ */
    
    .btn {
      padding: 0.75rem 1.5rem;
      border: none;
      border-radius: 8px;
      font-size: 1rem;
      font-weight: 600;
      cursor: pointer;
      transition: all 0.3s;
      text-align: center;
    }
    
    .btn-primario {
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      color: white;
      width: 100%;
    }
    
    .btn-primario:hover {
      transform: translateY(-2px);
      box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4);
    }
    
    .btn-primario:disabled {
      opacity: 0.6;
      cursor: not-allowed;
      transform: none;
    }
    
    .btn-secundario {
      background: #f5f5f5;
      color: #333;
    }
    
    .btn-secundario:hover {
      background: #e0e0e0;
    }
    
    .btn-finalizar {
      background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
      color: white;
    }
    
    /* ============================================ */
    /* MENSAJES */
    /* ============================================ */
    
    .mensaje-error {
      background: #ffebee;
      color: #c62828;
      padding: 1rem;
      border-radius: 8px;
      border-left: 4px solid #c62828;
      margin-top: 1rem;
      display: flex;
      align-items: center;
      gap: 0.5rem;
    }
    
    .mensaje-loading {
      background: #e3f2fd;
      color: #1565c0;
      padding: 1rem;
      border-radius: 8px;
      border-left: 4px solid #1565c0;
      margin-top: 1rem;
      display: flex;
      align-items: center;
      gap: 0.5rem;
    }
    
    /* ============================================ */
    /* CONTENEDOR PRINCIPAL DEL TEST */
    /* ============================================ */
    
    .contenedor-test {
      background: white;
      width: 95%;
      max-width: 1200px;
      min-height: 90vh;
      border-radius: 12px;
      box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
      overflow: hidden;
      display: none;
    }
    
    .header-test {
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      color: white;
      padding: 1.5rem;
      display: flex;
      justify-content: space-between;
      align-items: center;
      flex-wrap: wrap;
      gap: 1rem;
    }
    
    .header-test h1 {
      font-size: 1.5rem;
    }
    
    .info-estudiante {
      display: flex;
      gap: 2rem;
      flex-wrap: wrap;
    }
    
    .info-estudiante span {
      display: flex;
      align-items: center;
      gap: 0.5rem;
    }
    
    /* ============================================ */
    /* INSTRUCCIONES */
    /* ============================================ */
    
    .contenedor-instrucciones {
      background: white;
      width: 95%;
      max-width: 900px;
      padding: 3rem;
      border-radius: 12px;
      box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
      display: none;
    }
    
    .contenedor-instrucciones h1 {
      color: #667eea;
      margin-bottom: 0.5rem;
    }
    
    .contenedor-instrucciones h2 {
      color: #333;
      margin-bottom: 1.5rem;
    }
    
    .instrucciones-contenido {
      color: #555;
      line-height: 1.8;
      margin-bottom: 2rem;
    }
    
    .instrucciones-contenido ul {
      list-style: none;
      padding-left: 0;
    }
    
    .instrucciones-contenido li {
      padding: 0.5rem 0;
      padding-left: 1.5rem;
      position: relative;
    }
    
    .instrucciones-contenido li:before {
      content: "✓";
      position: absolute;
      left: 0;
      color: #667eea;
      font-weight: bold;
    }
    
    .competencias {
      background: #f5f5f5;
      padding: 1.5rem;
      border-radius: 8px;
      margin-top: 1.5rem;
    }
    
    .competencias h3 {
      color: #333;
      margin-bottom: 1rem;
    }
    
    /* ============================================ */
    /* BARRA DE PROGRESO */
    /* ============================================ */
    
    .progreso {
      padding: 1.5rem;
      background: #f9f9f9;
      border-bottom: 1px solid #e0e0e0;
    }
    
    .progreso-info {
      display: flex;
      justify-content: space-between;
      margin-bottom: 0.75rem;
      font-weight: 600;
      color: #555;
    }
    
    .barra-progreso {
      width: 100%;
      height: 8px;
      background: #e0e0e0;
      border-radius: 10px;
      overflow: hidden;
    }
    
    .progreso-actual {
      height: 100%;
      background: linear-gradient(90deg, #667eea 0%, #764ba2 100%);
      transition: width 0.3s ease;
      border-radius: 10px;
    }
    
    /* ============================================ */
    /* PREGUNTA */
    /* ============================================ */
    
    .pregunta-contenedor {
      padding: 2rem;
      max-width: 900px;
      margin: 0 auto;
    }
    
    .pregunta-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 1.5rem;
      flex-wrap: wrap;
      gap: 1rem;
    }
    
    .numero-pregunta {
      font-size: 1.25rem;
      font-weight: bold;
      color: #667eea;
    }
    
    .etiqueta-competencia {
      background: #e8eaf6;
      color: #667eea;
      padding: 0.4rem 1rem;
      border-radius: 20px;
      font-size: 0.85rem;
      font-weight: 600;
    }
    
    .contexto {
      background: #f5f5f5;
      padding: 1.5rem;
      border-radius: 8px;
      margin-bottom: 1.5rem;
      line-height: 1.8;
      color: #333;
    }
    
    .contenedor-grafico {
      margin: 1.5rem 0;
      padding: 1rem;
      background: #fafafa;
      border-radius: 8px;
      border: 1px solid #e0e0e0;
    }
    
    .contenedor-grafico canvas {
      max-height: 400px;
    }
    
    .pregunta-texto {
      font-size: 1.1rem;
      font-weight: 600;
      color: #333;
      margin: 1.5rem 0;
      line-height: 1.6;
    }
    
    /* ============================================ */
    /* OPCIONES DE RESPUESTA */
    /* ============================================ */
    
    .opciones {
      display: flex;
      flex-direction: column;
      gap: 1rem;
    }
    
    .opcion {
      display: flex;
      align-items: flex-start;
      padding: 1rem 1.25rem;
      border: 2px solid #e0e0e0;
      border-radius: 8px;
      cursor: pointer;
      transition: all 0.3s;
      background: white;
    }
    
    .opcion:hover {
      border-color: #667eea;
      background: #f5f7ff;
    }
    
    .opcion.seleccionada {
      border-color: #667eea;
      background: #e8eaf6;
    }
    
    .opcion input[type="radio"] {
      display: none;
    }
    
    .letra-opcion {
      display: inline-flex;
      align-items: center;
      justify-content: center;
      width: 32px;
      height: 32px;
      border-radius: 50%;
      background: #f5f5f5;
      color: #666;
      font-weight: bold;
      margin-right: 1rem;
      flex-shrink: 0;
      transition: all 0.3s;
    }
    
    .opcion.seleccionada .letra-opcion {
      background: #667eea;
      color: white;
    }
    
    .texto-opcion {
      flex: 1;
      line-height: 1.6;
      color: #333;
    }
    
    /* ============================================ */
    /* NAVEGACIÓN */
    /* ============================================ */
    
    .navegacion {
      padding: 1.5rem 2rem;
      border-top: 1px solid #e0e0e0;
      display: flex;
      justify-content: space-between;
      gap: 1rem;
      flex-wrap: wrap;
    }
    
    .navegacion button {
      padding: 0.75rem 2rem;
    }
    
    /* ============================================ */
    /* MAPA DE PREGUNTAS */
    /* ============================================ */
    
    .mapa-preguntas {
      padding: 1.5rem 2rem;
      background: #fafafa;
      border-top: 1px solid #e0e0e0;
    }
    
    .mapa-preguntas h4 {
      margin-bottom: 1rem;
      color: #555;
    }
    
    .mapa-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(50px, 1fr));
      gap: 0.5rem;
      margin-bottom: 1rem;
    }
    
    .btn-mapa {
      width: 50px;
      height: 50px;
      border: 2px solid #e0e0e0;
      background: white;
      border-radius: 8px;
      cursor: pointer;
      font-weight: 600;
      transition: all 0.3s;
    }
    
    .btn-mapa:hover {
      border-color: #667eea;
      transform: scale(1.05);
    }
    
    .btn-mapa.respondida {
      background: #4caf50;
      color: white;
      border-color: #4caf50;
    }
    
    .btn-mapa.actual {
      background: #667eea;
      color: white;
      border-color: #667eea;
    }
    
    .leyenda-mapa {
      display: flex;
      gap: 2rem;
      flex-wrap: wrap;
      font-size: 0.9rem;
      color: #666;
    }
    
    .leyenda-mapa span {
      display: flex;
      align-items: center;
      gap: 0.5rem;
    }
    
    .indicador {
      width: 20px;
      height: 20px;
      border-radius: 4px;
      border: 2px solid #e0e0e0;
    }
    
    .indicador.respondida {
      background: #4caf50;
      border-color: #4caf50;
    }
    
    .indicador.pendiente {
      background: white;
    }
    
    /* ============================================ */
    /* MODAL DE ÉXITO */
    /* ============================================ */
    
    .exito-icono {
      font-size: 4rem;
      text-align: center;
      margin-bottom: 1rem;
    }
    
    .modal-acciones {
      display: flex;
      gap: 1rem;
      margin-top: 1.5rem;
    }
    
    .modal-acciones button {
      flex: 1;
    }
    
    .resumen-respuestas {
      background: #f5f5f5;
      padding: 1rem;
      border-radius: 8px;
      margin: 1rem 0;
    }
    
    .resumen-respuestas p {
      margin: 0.5rem 0;
    }
    
    /* ============================================ */
    /* RESPONSIVE */
    /* ============================================ */
    
    @media (max-width: 768px) {
      .header-test {
        flex-direction: column;
        align-items: flex-start;
      }
      
      .mapa-grid {
        grid-template-columns: repeat(auto-fill, minmax(45px, 1fr));
      }
      
      .navegacion {
        flex-direction: column;
      }
      
      .navegacion button {
        width: 100%;
      }
    }
  </style>
</head>
<body>

  <!-- ============================================ -->
  <!-- MODAL DE AUTENTICACIÓN -->
  <!-- ============================================ -->
  <div id="modal-autenticacion" class="modal-overlay">
    <div class="modal-contenido">
      <h2>🎓 Simulacro ICFES - Matemáticas</h2>
      <p>Ingresa tu número de documento para acceder al test</p>
      
      <form id="form-autenticacion">
        <div class="form-group">
          <label for="documento">Número de Documento</label>
          <input 
            type="text" 
            id="documento" 
            placeholder="Ej: 1234567890" 
            required
            pattern="[0-9]+"
            title="Solo números"
            maxlength="15"
          >
        </div>
        <button type="submit" class="btn btn-primario">Ingresar</button>
      </form>
      
      <div id="error-autenticacion" class="mensaje-error" style="display:none;">
        <span>❌</span>
        <span>Documento no encontrado. Contacta al administrador.</span>
      </div>
      
      <div id="loading-autenticacion" class="mensaje-loading" style="display:none;">
        <span>🔄</span>
        <span>Verificando documento...</span>
      </div>
    </div>
  </div>

  <!-- ============================================ -->
  <!-- PANTALLA DE INSTRUCCIONES -->
  <!-- ============================================ -->
  <div id="instrucciones" class="contenedor-instrucciones">
    <h1>¡Bienvenido/a, <span id="nombre-estudiante"></span>!</h1>
    <h2>Instrucciones del Simulacro de Matemáticas</h2>
    
    <div class="instrucciones-contenido">
      <p><strong>Esta prueba contiene 30 preguntas de selección múltiple con única respuesta.</strong></p>
      
      <ul>
        <li>Lee cuidadosamente cada pregunta y su contexto</li>
        <li>Algunas preguntas incluyen gráficos, tablas o diagramas</li>
        <li>Selecciona UNA opción de respuesta (A, B, C o D)</li>
        <li>Puedes navegar entre preguntas usando los botones</li>
        <li>El mapa inferior te permite ir directamente a cualquier pregunta</li>
        <li>Asegúrate de responder todas las preguntas antes de finalizar</li>
        <li><strong>Una vez envíes tus respuestas, no podrás modificarlas</strong></li>
      </ul>
      
      <div class="competencias">
        <h3>📊 Competencias evaluadas:</h3>
        <ul>
          <li><strong>Comunicación, representación y modelación:</strong> Interpretación de información matemática</li>
          <li><strong>Planteamiento y resolución de problemas:</strong> Aplicación de conceptos a situaciones</li>
          <li><strong>Razonamiento y argumentación:</strong> Justificación de procedimientos</li>
        </ul>
      </div>
    </div>
    
    <button id="btn-comenzar" class="btn btn-primario">🚀 Comenzar Simulacro</button>
  </div>

  <!-- ============================================ -->
  <!-- CONTENEDOR PRINCIPAL DEL TEST -->
  <!-- ============================================ -->
  <div id="contenedor-test" class="contenedor-test">
    
    <!-- Header -->
    <div class="header-test">
      <h1>📐 Simulacro ICFES - Matemáticas</h1>
      <div class="info-estudiante">
        <span>👤 <strong id="header-nombre"></strong></span>
        <span>🆔 <strong id="header-documento"></strong></span>
      </div>
    </div>

    <!-- Barra de progreso -->
    <div class="progreso">
      <div class="progreso-info">
        <span id="contador-preguntas">Pregunta 1 de 30</span>
        <span id="contador-respondidas">Respondidas: 0/30</span>
      </div>
      <div class="barra-progreso">
        <div class="progreso-actual" style="width: 0%;"></div>
      </div>
    </div>

    <!-- Pregunta actual -->
    <div id="pregunta-actual" class="pregunta-contenedor">
      <!-- Se renderiza dinámicamente con JavaScript -->
    </div>

    <!-- Navegación -->
    <div class="navegacion">
      <button id="btn-anterior" class="btn btn-secundario" disabled>← Anterior</button>
      <button id="btn-siguiente" class="btn btn-primario">Siguiente →</button>
      <button id="btn-finalizar" class="btn btn-finalizar" style="display:none;">✓ Finalizar Test</button>
    </div>

    <!-- Mapa de preguntas -->
    <div class="mapa-preguntas">
      <h4>Navegación rápida:</h4>
      <div id="mapa-botones" class="mapa-grid">
        <!-- Botones 1-30 generados dinámicamente -->
      </div>
      <div class="leyenda-mapa">
        <span><span class="indicador respondida"></span> Respondida</span>
        <span><span class="indicador pendiente"></span> Pendiente</span>
      </div>
    </div>

  </div>

  <!-- ============================================ -->
  <!-- MODAL DE CONFIRMACIÓN -->
  <!-- ============================================ -->
  <div id="modal-confirmacion" class="modal-overlay" style="display:none;">
    <div class="modal-contenido">
      <h2>⚠️ ¿Finalizar el simulacro?</h2>
      
      <div class="resumen-respuestas">
        <p>Has respondido <strong id="total-respondidas">0</strong> de <strong>30</strong> preguntas.</p>
        <p id="advertencia-pendientes" style="display:none; color: #d32f2f; font-weight: 600;">
          ⚠️ Tienes <strong id="num-pendientes"></strong> preguntas sin responder.
        </p>
      </div>
      
      <p style="margin-top: 1rem;">Una vez envíes, no podrás modificar tus respuestas.</p>
      
      <div class="modal-acciones">
        <button id="btn-continuar" class="btn btn-secundario">← Revisar respuestas</button>
        <button id="btn-confirmar-envio" class="btn btn-finalizar">Sí, enviar respuestas →</button>
      </div>
    </div>
  </div>

  <!-- ============================================ -->
  <!-- MODAL DE ÉXITO -->
  <!-- ============================================ -->
  <div id="modal-exito" class="modal-overlay" style="display:none;">
    <div class="modal-contenido">
      <div class="exito-icono">✅</div>
      <h2>¡Respuestas enviadas correctamente!</h2>
      <p>Gracias por completar el simulacro, <strong id="exito-nombre"></strong>.</p>
      <p>Tus resultados serán procesados y estarán disponibles próximamente.</p>
      <button class="btn btn-primario" onclick="location.reload()">Cerrar</button>
    </div>
  </div>

  <script>
    // ============================================
    // ⚙️ CONFIGURACIÓN DE SUPABASE
    // ============================================
    const SUPABASE_URL = 'TU_URL_DE_SUPABASE';
    const SUPABASE_ANON_KEY = 'TU_ANON_KEY_DE_SUPABASE';

    // Inicializar cliente de Supabase
    const { createClient } = supabase;
    const supabaseClient = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);

    // ============================================
    // VARIABLES GLOBALES
    // ============================================
    let estudianteActual = {
      documento: null,
      nombre: ''
    };

    let estadoTest = {
      preguntaActual: 0,
      respuestas: {}, // { numeroPregunta: 'A' }
      horaInicio: null
    };

    // ============================================
    // BANCO DE PREGUNTAS (30 preguntas)
    // ============================================
    const bancoPreguntas = [
      // AQUÍ SE GENERAN LAS 30 PREGUNTAS
      // Ejemplo de estructura:
      /*
      {
        numero: 1,
        competencia: "Razonamiento cuantitativo",
        componente: "Numérico-variacional",
        tema: "Proporcionalidad",
        dificultad: "media",
        contexto: "Una empresa de...",
        tiene_grafico: true,
        tipo_grafico: "tabla",
        datos_grafico: {...},
        pregunta: "¿Cuál es...?",
        opciones: {
          A: "Opción A",
          B: "Opción B",
          C: "Opción C",
          D: "Opción D"
        },
        respuesta_correcta: "B",
        justificacion: "Porque..."
      }
      */
    ];

    // ============================================
    // AUTENTICACIÓN
    // ============================================
    document.getElementById('form-autenticacion').addEventListener('submit', async (e) => {
      e.preventDefault();
      
      const documento = parseInt(document.getElementById('documento').value.trim());
      const errorDiv = document.getElementById('error-autenticacion');
      const loadingDiv = document.getElementById('loading-autenticacion');
      
      // Mostrar loading
      errorDiv.style.display = 'none';
      loadingDiv.style.display = 'flex';

      try {
        // Consultar tabla estudiantes en Supabase
        const { data, error } = await supabaseClient
          .from('estudiantes')
          .select('documento, nombre')
          .eq('documento', documento)
          .single();

        loadingDiv.style.display = 'none';

        if (error || !data) {
          // Documento no encontrado
          errorDiv.style.display = 'flex';
          return;
        }

        // Estudiante encontrado
        estudianteActual.documento = data.documento;
        estudianteActual.nombre = data.nombre;

        // Ocultar modal, mostrar instrucciones
        document.getElementById('modal-autenticacion').style.display = 'none';
        document.getElementById('nombre-estudiante').textContent = estudianteActual.nombre;
        document.getElementById('instrucciones').style.display = 'flex';

      } catch (error) {
        console.error('Error al consultar Supabase:', error);
        loadingDiv.style.display = 'none';
        errorDiv.innerHTML = '<span>❌</span><span>Error de conexión. Intenta nuevamente.</span>';
        errorDiv.style.display = 'flex';
      }
    });

    // ============================================
    // INICIAR TEST
    // ============================================
    document.getElementById('btn-comenzar').addEventListener('click', () => {
      estadoTest.horaInicio = new Date();
      
      // Actualizar header
      document.getElementById('header-nombre').textContent = estudianteActual.nombre;
      document.getElementById('header-documento').textContent = estudianteActual.documento;

      // Ocultar instrucciones, mostrar test
      document.getElementById('instrucciones').style.display = 'none';
      document.getElementById('contenedor-test').style.display = 'block';

      // Generar mapa de navegación
      generarMapaPreguntas();

      // Renderizar primera pregunta
      renderizarPregunta(0);
    });

    // ============================================
    // RENDERIZAR PREGUNTA
    // ============================================
    function renderizarPregunta(index) {
      const pregunta = bancoPreguntas[index];
      const contenedor = document.getElementById('pregunta-actual');

      let htmlGrafico = '';
      if (pregunta.tiene_grafico) {
        if (pregunta.tipo_grafico === 'tabla') {
          htmlGrafico = generarTabla(pregunta.datos_grafico);
        } else {
          htmlGrafico = `<div class="contenedor-grafico">
            <canvas id="grafico-pregunta-${pregunta.numero}"></canvas>
          </div>`;
        }
      }

      contenedor.innerHTML = `
        <div class="pregunta-header">
          <span class="numero-pregunta">Pregunta ${pregunta.numero}</span>
          <span class="etiqueta-competencia">${pregunta.competencia}</span>
        </div>

        <div class="contexto">${pregunta.contexto}</div>

        ${htmlGrafico}

        <div class="pregunta-texto">${pregunta.pregunta}</div>

        <div class="opciones">
          ${Object.entries(pregunta.opciones).map(([letra, texto]) => `
            <label class="opcion ${estadoTest.respuestas[pregunta.numero] === letra ? 'seleccionada' : ''}">
              <input 
                type="radio" 
                name="respuesta-${pregunta.numero}" 
                value="${letra}"
                ${estadoTest.respuestas[pregunta.numero] === letra ? 'checked' : ''}
              >
              <span class="letra-opcion">${letra}</span>
              <span class="texto-opcion">${texto}</span>
            </label>
          `).join('')}
        </div>
      `;

      // Renderizar gráfico si existe (y no es tabla)
      if (pregunta.tiene_grafico && pregunta.tipo_grafico !== 'tabla') {
        setTimeout(() => renderizarGrafico(pregunta), 100);
      }

      // Event listeners para opciones
      document.querySelectorAll(`input[name="respuesta-${pregunta.numero}"]`).forEach(input => {
        input.addEventListener('change', (e) => {
          estadoTest.respuestas[pregunta.numero] = e.target.value;
          
          // Actualizar visualmente
          document.querySelectorAll('.opcion').forEach(op => op.classList.remove('seleccionada'));
          e.target.closest('.opcion').classList.add('seleccionada');
          
          actualizarEstado();
        });
      });

      // Actualizar navegación
      actualizarNavegacion(index);
      actualizarEstado();
    }

    // ============================================
    // GENERAR TABLA HTML
    // ============================================
    function generarTabla(datos) {
      // datos = { headers: [...], rows: [[...], [...]] }
      return `
        <div class="contenedor-grafico">
          <table style="width: 100%; border-collapse: collapse;">
            <thead>
              <tr>
                ${datos.headers.map(h => `<th style="border: 1px solid #ddd; padding: 12px; background: #f5f5f5; font-weight: 600;">${h}</th>`).join('')}
              </tr>
            </thead>
            <tbody>
              ${datos.rows.map(row => `
                <tr>
                  ${row.map(cell => `<td style="border: 1px solid #ddd; padding: 12px; text-align: center;">${cell}</td>`).join('')}
                </tr>
              `).join('')}
            </tbody>
          </table>
        </div>
      `;
    }

    // ============================================
    // RENDERIZAR GRÁFICOS (Chart.js)
    // ============================================
    function renderizarGrafico(pregunta) {
      const ctx = document.getElementById(`grafico-pregunta-${pregunta.numero}`);
      if (!ctx) return;

      // Crear gráfico según tipo
      new Chart(ctx.getContext('2d'), pregunta.configuracion_grafico);
    }

    // ============================================
    // NAVEGACIÓN
    // ============================================
    document.getElementById('btn-siguiente').addEventListener('click', () => {
      if (estadoTest.preguntaActual < bancoPreguntas.length - 1) {
        estadoTest.preguntaActual++;
        renderizarPregunta(estadoTest.preguntaActual);
      }
    });

    document.getElementById('btn-anterior').addEventListener('click', () => {
      if (estadoTest.preguntaActual > 0) {
        estadoTest.preguntaActual--;
        renderizarPregunta(estadoTest.preguntaActual);
      }
    });

    // ============================================
    // FINALIZAR TEST
    // ============================================
    document.getElementById('btn-finalizar').addEventListener('click', () => {
      const respondidas = Object.keys(estadoTest.respuestas).length;
      const pendientes = bancoPreguntas.length - respondidas;

      document.getElementById('total-respondidas').textContent = respondidas;
      
      if (pendientes > 0) {
        document.getElementById('num-pendientes').textContent = pendientes;
        document.getElementById('advertencia-pendientes').style.display = 'block';
      } else {
        document.getElementById('advertencia-pendientes').style.display = 'none';
      }

      document.getElementById('modal-confirmacion').style.display = 'flex';
    });

    document.getElementById('btn-continuar').addEventListener('click', () => {
      document.getElementById('modal-confirmacion').style.display = 'none';
    });

    document.getElementById('btn-confirmar-envio').addEventListener('click', async () => {
      await enviarRespuestas();
    });

    // ============================================
    // ENVÍO A SUPABASE
    // ============================================
    async function enviarRespuestas() {
        // Preparar objeto de respuestas en formato JSON
        const objetoRespuestas = {};
        bancoPreguntas.forEach(pregunta => {
            objetoRespuestas[pregunta.numero] = estadoTest.respuestas[pregunta.numero] || 'SIN_RESPUESTA';
        });

        // Preparar registro único
        const registro = {
            documento_estudiante: estudianteActual.documento,
            respuestas: objetoRespuestas
        };

        try {
            const { data, error } = await supabaseClient
                    .from('respuestas_estudiantes')
                    .insert([registro]);

            if (error) throw error;

            // Mostrar éxito
            document.getElementById('modal-confirmacion').style.display = 'none';
            document.getElementById('exito-nombre').textContent = estudianteActual.nombre;
            document.getElementById('modal-exito').style.display = 'flex';

        } catch (error) {
            console.error('Error al enviar respuestas:', error);
            alert('❌ Error al enviar respuestas. Por favor, intenta nuevamente.\n\n' + error.message);
        }
    }

    // ============================================
    // FUNCIONES AUXILIARES
    // ============================================
    function actualizarEstado() {
      const respondidas = Object.keys(estadoTest.respuestas).length;
      const progreso = (respondidas / bancoPreguntas.length) * 100;

      document.getElementById('contador-preguntas').textContent = 
        `Pregunta ${estadoTest.preguntaActual + 1} de ${bancoPreguntas.length}`;
      document.getElementById('contador-respondidas').textContent = 
        `Respondidas: ${respondidas}/${bancoPreguntas.length}`;
      document.querySelector('.progreso-actual').style.width = `${progreso}%`;

      // Actualizar mapa
      actualizarMapaPreguntas();
    }

    function generarMapaPreguntas() {
      const mapa = document.getElementById('mapa-botones');
      mapa.innerHTML = bancoPreguntas.map((p, i) => 
        `<button class="btn-mapa" data-index="${i}">${p.numero}</button>`
      ).join('');

      mapa.querySelectorAll('.btn-mapa').forEach(btn => {
        btn.addEventListener('click', (e) => {
          const index = parseInt(e.target.dataset.index);
          estadoTest.preguntaActual = index;
          renderizarPregunta(index);
        });
      });
    }

    function actualizarMapaPreguntas() {
      document.querySelectorAll('.btn-mapa').forEach((btn, i) => {
        const numeroPregunta = bancoPreguntas[i].numero;
        btn.classList.remove('respondida', 'actual');
        
        if (estadoTest.respuestas[numeroPregunta]) {
          btn.classList.add('respondida');
        }
        
        if (i === estadoTest.preguntaActual) {
          btn.classList.add('actual');
        }
      });
    }

    function actualizarNavegacion(index) {
      document.getElementById('btn-anterior').disabled = (index === 0);
      
      if (index === bancoPreguntas.length - 1) {
        document.getElementById('btn-siguiente').style.display = 'none';
        document.getElementById('btn-finalizar').style.display = 'inline-block';
      } else {
        document.getElementById('btn-siguiente').style.display = 'inline-block';
        document.getElementById('btn-finalizar').style.display = 'none';
      }
    }

  </script>

</body>
</html>
```

##Prevenir Cierre Accidental del Navegador 🚨
Problema: Estudiante cierra por error y pierde todo.
Solución en el HTML:
```
javascript
// ============================================
// PREVENCIÓN DE CIERRE ACCIDENTAL
// ============================================
let respuestasEnviadas = false;

window.addEventListener('beforeunload', (e) => {
  if (!respuestasEnviadas && Object.keys(estadoTest.respuestas).length > 0) {
    e.preventDefault();
    e.returnValue = '⚠️ ¿Seguro que quieres salir? Perderás todas tus respuestas.';
    return e.returnValue;
  }
});

// En la función enviarRespuestas, después de éxito.:
respuestasEnviadas = true;
```

---

## Integración con Supabase

### ✅ Tabla `estudiantes` (ya existente)
```sql
CREATE TABLE estudiantes (
  documento INT8 PRIMARY KEY,
  nombre VARCHAR NOT NULL
);
```

### ✅ Tabla `respuestas_estudiantes` (ya existente)
```sql
CREATE TABLE respuestas_estudiantes (
  id SERIAL PRIMARY KEY,
  documento_estudiante INT8 REFERENCES estudiantes(documento),
  respuestas JSONB NOT NULL
);
```

### 📤 Formato de envío a Supabase

Para cada estudiante, se inserta **UN SOLO registro** con todas las respuestas en formato JSON:
```javascript
// Preparar objeto de respuestas
const objetoRespuestas = {};
bancoPreguntas.forEach(pregunta => {
  objetoRespuestas[pregunta.numero] = estadoTest.respuestas[pregunta.numero] || 'SIN_RESPUESTA';
});

const registro = {
  documento_estudiante: estudianteActual.documento,
  respuestas: objetoRespuestas
};

try {
  const { data, error } = await supabaseClient
    .from('respuestas_estudiantes')
    .insert([registro]);

  if (error) throw error;

  // Mostrar éxito
  document.getElementById('modal-confirmacion').style.display = 'none';
  document.getElementById('exito-nombre').textContent = estudianteActual.nombre;
  document.getElementById('modal-exito').style.display = 'flex';

} catch (error) {
  console.error('Error al enviar respuestas:', error);
  alert('❌ Error al enviar respuestas. Por favor, intenta nuevamente.\n\n' + error.message);
}
```

---

## Análisis de Resultados (Próxima Fase)

Una vez el usuario adjunte un documento con:
```
| documento_estudiante | respuestas      | 
|----------------------|-----------------|
| 1234567890           | JSON            |
| 1234567890           | JSON            |
```

Deberás generar un **análisis de falencias** que incluya:

### 📊 Indicadores Clave
1. **Puntaje global**: X/30 preguntas correctas (Y%)
2. **Desempeño por competencia**:
    - Razonamiento: X/10 correctas
    - Modelación: X/10 correctas
    - Resolución de problemas: X/10 correctas
3. **Desempeño por componente**:
    - Numérico-variacional: X%
    - Geométrico-métrico: X%
    - Aleatorio: X%
4. **Preguntas falladas**: Listado con justificación
5. **Recomendaciones**: Temas específicos a reforzar

---

## Criterios de Calidad

### ✅ Diseño y UX
- Interfaz limpia, profesional, similar al diseño oficial ICFES
- Responsive (funcional en desktop, tablet, móvil)
- Tipografía legible (mínimo 16px)
- Contraste adecuado
- Animaciones suaves

### ✅ Contenido de Preguntas
- Contextos realistas y variados
- Redacción clara
- Opciones homogéneas
- Distractores plausibles
- Balance de dificultad: 40% fácil, 40% medio, 20% difícil

### ✅ Gráficos
- Datos coherentes
- Ejes etiquetados
- Leyendas claras
- Colores armoniosos
- Tamaño adecuado

---

## Entregables

1. **Artefacto HTML funcional** con:
    - Modal de autenticación con Supabase
    - 30 preguntas (10 con gráficos)
    - Navegación completa
    - Envío a Supabase mediante librería oficial

2. **Archivo JSON** con el banco de preguntas estructurado

3. **Comentario en el código** indicando donde configurar:
```javascript
   const SUPABASE_URL = 'TU_URL_AQUÍ';
   const SUPABASE_ANON_KEY = 'TU_KEY_AQUÍ';
```

---

## Restricciones

❌ NO usar localStorage/sessionStorage  
❌ NO importar imágenes externas  
❌ NO copiar preguntas literales  
✅ USAR librería oficial de Supabase  
✅ GENERAR gráficos con código  
✅ INNOVAR sobre patrones ICFES

---

##Extra
- Todas las preguntas son obligatorias. no se puede enviar las respuestas a supabase si alguna pregunta no está respondida. Entonces antes de finalizar y enviar respuestas se debe verificar y obligar a marcar todas las preguntas.

- Cuando te envie varios consolidados de respuestas de respondidas por los estudiantes te pediré un informe que abarque de forma general como les fue al grupo completo. Cuales fueron sus falencias, que deben reforzar y por otro lado hay que exhaltar lo positivo, así que también en el informe señalaras que fortalezas encontraste (si las hay).

---

¿Tienes acceso a los archivos adjuntos? Una vez los tengas, genera el artefacto HTML completo con las 30 preguntas. 🚀