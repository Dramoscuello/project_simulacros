# Generador de Banco de Preguntas ICFES - Matemáticas

## Contexto del Proyecto

En Colombia se realizan anualmente las **Pruebas de Estado ICFES (Saber 11°)** para estudiantes de grado 11, evaluando competencias en las áreas de Lectura Crítica, Matemáticas, Sociales y Ciudadanas, Ciencias Naturales e Inglés.

Te adjunto:
- 📘 Guía oficial del Ministerio de Educación Nacional
- Los niveles de desempeño en matemáticas
- Marco de referencia
- 📄 Pruebas de matemáticas de años anteriores (PDF)

## Objetivo Principal

Analizar los materiales adjuntos y **generar un banco de preguntas originales** para el área de **Matemáticas**, siguiendo la guía oficial del ministerio de educación nacional, los patrones, marco de referencia y estándares del ICFES, pero con innovación en los contextos, datos y situaciones planteadas.

**⚠️ IMPORTANTE**: No se trata de replicar preguntas existentes, sino de **innovar manteniendo la estructura, competencias, niveles de desempeño y nivel de dificultad** observados en las pruebas oficiales.

---

## Sistema Multi-tenant

El sistema soporta múltiples instituciones educativas. Cada simulacro se asigna a una institución específica para:
- Evitar repetición de preguntas por institución
- Análisis de resultados por institución
- Trazabilidad de versiones de simulacros

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

### Tabla `instituciones`
```sql
CREATE TABLE instituciones (
  id SERIAL PRIMARY KEY,
  nombre VARCHAR(255) NOT NULL,
  direccion VARCHAR(100),
  activo BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW()
);
```

**Ejemplo de datos:**
```
| id                                   | nombre              | activo |
|--------------------------------------|---------------------|--------|
| 1                                    | Colegio San José    | true   |
| 2                                    | IE La Esperanza     | true   |
```

### Tabla `estudiantes`
```sql
CREATE TABLE estudiantes (
  documento BIGINT PRIMARY KEY,
  nombre VARCHAR(255) NOT NULL,
  institucion_id INT REFERENCES instituciones(id) ON DELETE CASCADE,
  admin BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT NOW()
);
```

**Ejemplo de datos:**
```
| documento  | nombre              | institucion_id                       | admin |
|------------|---------------------|--------------------------------------|-------|
| 1234567890 | Juan Pérez García   | 1                                    | false |
| 1063162459 | DEIMER Admin        | 2                                    | true  |
```

### Tabla `preguntas_usadas`
```sql
CREATE TABLE preguntas_usadas (
  id SERIAL PRIMARY KEY,
  institucion_id INT REFERENCES instituciones(id) ON DELETE CASCADE,
  pregunta TEXT NOT NULL,
  tema VARCHAR(255),
  componente VARCHAR(255),
  competencia VARCHAR(255),
  version_simulacro VARCHAR(50),
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_preguntas_institucion ON preguntas_usadas(institucion_id);
```

**Ejemplo de datos:**
```
| id | institucion_id                       | pregunta                        | tema                    | version_simulacro |
|----|--------------------------------------|---------------------------------|-------------------------|-------------------|
| 1  | 1                                    | Si 3x + 5 = 20, ¿cuál es x?    | Álgebra - Ecuaciones    | v2025-01          |
| 2  | 2                                    | Un rectángulo tiene 12cm...     | Geometría - Áreas       | v2025-01          |
```

### Tabla `respuestas_estudiantes`
```sql
CREATE TABLE respuestas_estudiantes (
  id SERIAL PRIMARY KEY,
  documento_estudiante BIGINT REFERENCES estudiantes(documento),
  respuestas JSONB NOT NULL,
  respuestas_detalladas JSONB NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  notificado BOOLEAN DEFAULT false
);

CREATE INDEX idx_respuestas_institucion ON respuestas_estudiantes(institucion_id);
```

**Ejemplo de datos después del envío:**
```json
{
  "id": 221,
  "documento_estudiante": 1063162459,
  "respuestas": {
    "1": "B",
    "2": "A",
    "3": "C"
  },
  "respuestas_detalladas": {
    "1": {
      "respuesta": "B",
      "competencia": "Razonamiento y argumentación",
      "componente": "Numérico-variacional",
      "tema": "Álgebra - Ecuaciones lineales",
      "acierto": true,
      "respuesta_correcta": "B"
    },
    "2": {
      "respuesta": "A",
      "competencia": "Planteamiento y resolución de problemas",
      "componente": "Geométrico-métrico",
      "tema": "Geometría - Áreas y perímetros",
      "acierto": false,
      "respuesta_correcta": "C"
    },
    "3": {
      "respuesta": "C",
      "competencia": "Comunicación, representación y modelación",
      "componente": "Aleatorio",
      "tema": "Estadística - Probabilidad",
      "acierto": true,
      "respuesta_correcta": "C"
    }
  },
  "notificado": false
}
```

---

## Especificaciones Técnicas

### 1. Estructura de las Preguntas

Cada pregunta debe incluir:
```json
{
  "numero": 1,
  "competencia": "Razonamiento y argumentación",
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

### IMPORTANTE: Incluir metadatos en cada pregunta

Cada pregunta DEBE incluir los siguientes campos para análisis:
- `competencia`: "Comunicación, representación y modelación" | "Planteamiento y resolución de problemas" | "Razonamiento y argumentación"
- `componente`: "Numérico-variacional" | "Geométrico-métrico" | "Aleatorio"
- `tema`: Tema específico (ej: "Álgebra - Sistemas de ecuaciones")
- `respuesta_correcta`: La opción correcta ("A", "B", "C" o "D")

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

## Generación de Preguntas sin Repetir

### Contexto del Usuario

**El usuario te proporcionará manualmente** un listado de preguntas que YA fueron usadas en simulacros anteriores para una institución específica. Este listado incluirá:
- Texto de la pregunta (primeras 100-150 palabras)
- Tema
- Componente
- Competencia
- Versión del simulacro

### SINO TE PROPORCIONA LISTADO DE PREGUNTAS ES QUE AUN NO SE HAN REALIZADO SIMULACROS ANTES EN ESA INSTITUCION EN ESPECIFICO

### Tu Responsabilidad

Cuando recibas este listado de preguntas ya usadas, debes:

1. **Analizar cuidadosamente** cada pregunta listada
2. **Identificar los temas, conceptos y enfoques** ya utilizados
3. **Generar 30 preguntas COMPLETAMENTE DIFERENTES** que:
    - Evalúen los **mismos temas** pero con **enfoques distintos**
    - Usen **contextos diferentes** (deportes, tecnología, economía, naturaleza, ciencia)
    - Tengan **valores numéricos distintos**
    - Presenten **variaciones en complejidad**
    - Utilicen **presentaciones variadas** (texto, gráficas, tablas)

### Ejemplo de Variación Correcta

**❌ PREGUNTA YA USADA:**
```
"Un rectángulo tiene 12 cm de largo y 8 cm de ancho. ¿Cuál es su área?"
Tema: Geometría - Áreas
```

**✅ PREGUNTA NUEVA (mismo tema, enfoque diferente):**
```
"Un terreno rectangular tiene un perímetro de 60 metros. Si el largo es el triple del ancho, ¿cuál es el área del terreno?"
Tema: Geometría - Áreas
```

**Ambas evalúan el mismo concepto (Geometría - Áreas) pero:**
- Contextos diferentes (figura geométrica vs terreno)
- Datos diferentes (medidas directas vs relación perímetro-dimensiones)
- Complejidad diferente (cálculo directo vs resolución de sistema)

### Instrucción Crítica

**Si el usuario te proporciona preguntas ya usadas:**
- Lee TODAS las preguntas cuidadosamente
- Identifica patrones: tipos de problemas, enfoques, contextos
- Genera preguntas ORIGINALES que:
    - NO repliquen estructuras existentes
    - NO usen contextos similares
    - NO tengan datos parecidos
    - PERO SÍ evalúen los mismos temas y competencias

---

## Artefacto HTML a Generar

Crea un **artefacto HTML completo** con las siguientes características:

### Sistema de Administrador

El artefacto debe incluir:

1. **Verificación de usuario administrador** al autenticarse
2. **Modal exclusivo para administradores** que permita:
    - Seleccionar la institución a la que pertenecen las preguntas del simulacro
    - Ingresar la versión del simulacro (ej: "v2025-01")
    - Guardar las 30 preguntas en la tabla `preguntas_usadas` de Supabase
3. **Opción de continuar sin guardar** (para testing)

### Flujo de Autenticación con Modo Admin
```javascript
// Después de autenticar con documento
const { data: estudiante } = await supabaseClient
  .from('estudiantes')
  .select('documento, nombre, admin, institucion_id')
  .eq('documento', documento)
  .single();

if (estudiante.admin === true) {
  // Mostrar modal de administrador
  mostrarModalAdmin();
} else {
  // Continuar flujo normal de estudiante
  mostrarInstrucciones();
}
```

### Modal de Administrador
```html
<!-- MODAL ADMINISTRADOR -->
<div id="modal-admin" class="modal-overlay" style="display:none;">
  <div class="modal-contenido">
    <h2>🔐 Modo Administrador</h2>
    <p>Configura las opciones del simulacro antes de continuar</p>
    
    <div class="form-group">
      <label for="institucion-select">Institución:</label>
      <select id="institucion-select" required>
        <option value="">-- Selecciona una institución --</option>
        <!-- Opciones cargadas dinámicamente desde Supabase -->
      </select>
    </div>
    
    <div class="form-group">
      <label for="version-simulacro">Versión del simulacro:</label>
      <input 
        type="text" 
        id="version-simulacro" 
        placeholder="Ej: v2025-01, enero-2025"
        required
      />
    </div>
    
    <div class="info-box">
      <strong>📝 Nota:</strong> Al confirmar, se guardarán las 30 preguntas de este simulacro 
      en la base de datos para evitar repetirlas en futuras versiones.
    </div>
    
    <button class="btn btn-primario" onclick="guardarPreguntasAdmin()">
      Confirmar y Guardar Preguntas
    </button>
    
    <button class="btn btn-secundario" onclick="continuarSinGuardar()">
      Continuar sin Guardar (Testing)
    </button>
  </div>
</div>
```

### Función para Guardar Preguntas
```javascript
async function guardarPreguntasAdmin() {
  const institucionId = document.getElementById('institucion-select').value;
  const versionSimulacro = document.getElementById('version-simulacro').value.trim();
  
  if (!institucionId || !versionSimulacro) {
    alert('Por favor completa todos los campos');
    return;
  }
  
  console.log('💾 Guardando preguntas en base de datos...');
  
  // Preparar array de preguntas para insertar
  const preguntasParaGuardar = bancoPreguntas.map(p => ({
    institucion_id: institucionId,
    pregunta: p.pregunta,
    tema: p.tema,
    componente: p.componente,
    competencia: p.competencia,
    version_simulacro: versionSimulacro
  }));
  
  try {
    const { data, error } = await supabaseClient
      .from('preguntas_usadas')
      .insert(preguntasParaGuardar);
    
    if (error) throw error;
    
    console.log('✅ Preguntas guardadas exitosamente');
    alert(`✅ ${bancoPreguntas.length} preguntas guardadas para la institución seleccionada.\n\nVersión: ${versionSimulacro}`);
    
    // Cerrar modal y continuar
    document.getElementById('modal-admin').style.display = 'none';
    mostrarInstrucciones();
    
  } catch (error) {
    console.error('❌ Error al guardar preguntas:', error);
    alert('Error al guardar preguntas. Revisa la consola para más detalles.');
  }
}

function continuarSinGuardar() {
  console.log('⚠️ Continuando sin guardar preguntas (modo testing)');
  document.getElementById('modal-admin').style.display = 'none';
  mostrarInstrucciones();
}

async function mostrarModalAdmin() {
  // Cargar instituciones activas desde Supabase
  const { data: instituciones, error } = await supabaseClient
    .from('instituciones')
    .select('id, nombre')
    .eq('activo', true)
    .order('nombre');
  
  if (error) {
    console.error('Error al cargar instituciones:', error);
    return;
  }
  
  // Llenar el select con las instituciones
  const select = document.getElementById('institucion-select');
  select.innerHTML = '<option value="">-- Selecciona una institución --</option>';
  
  instituciones.forEach(inst => {
    const option = document.createElement('option');
    option.value = inst.id;
    option.textContent = inst.nombre;
    select.appendChild(option);
  });
  
  // Generar versión automática basada en fecha
  const fecha = new Date();
  const mesAnio = `${fecha.getFullYear()}-${String(fecha.getMonth() + 1).padStart(2, '0')}`;
  document.getElementById('version-simulacro').value = `v${mesAnio}`;
  
  // Mostrar modal
  document.getElementById('modal-admin').style.display = 'flex';
}
```

### Estructura Completa del Artefacto
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
      max-width: 500px;
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
    
    .info-box {
      background: #eff6ff;
      border-left: 4px solid #2563eb;
      padding: 1rem;
      margin-top: 1rem;
      border-radius: 4px;
      font-size: 0.9rem;
      color: #1e40af;
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
    
    .form-group input,
    .form-group select {
      width: 100%;
      padding: 0.75rem;
      border: 2px solid #e0e0e0;
      border-radius: 8px;
      font-size: 1rem;
      transition: border-color 0.3s;
    }
    
    .form-group input:focus,
    .form-group select:focus {
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
      width: 100%;
      margin-top: 0.5rem;
    }
    
    .btn-primario {
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      color: white;
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
  <!-- MODAL ADMINISTRADOR -->
  <!-- ============================================ -->
  <div id="modal-admin" class="modal-overlay" style="display:none;">
    <div class="modal-contenido">
      <h2>🔐 Modo Administrador</h2>
      <p>Estás autenticado como administrador. Configura el simulacro:</p>
      
      <div class="form-group">
        <label for="institucion-select">Selecciona la institución:</label>
        <select id="institucion-select">
          <option value="">-- Selecciona una institución --</option>
        </select>
      </div>
      
      <div class="form-group">
        <label for="version-simulacro">Versión del simulacro:</label>
        <input 
          type="text" 
          id="version-simulacro" 
          placeholder="Ej: v2025-01, v2025-02"
        />
      </div>
      
      <div class="info-box">
        <strong>📝 Nota:</strong> Al confirmar, se guardarán las 30 preguntas de este simulacro 
        en la base de datos para evitar repetirlas en futuras versiones.
      </div>
      
      <button class="btn btn-primario" onclick="guardarPreguntasAdmin()">
        Confirmar y Guardar Preguntas
      </button>
      
      <button class="btn btn-secundario" onclick="continuarSinGuardar()">
        Continuar sin Guardar
      </button>
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
      nombre: '',
      esAdmin: false,
      institucionId: null
    };

    let estadoTest = {
      preguntaActual: 0,
      respuestas: {},
      horaInicio: null
    };

    // ============================================
    // BANCO DE PREGUNTAS (30 preguntas)
    // ============================================
    const bancoPreguntas = [
      // AQUÍ GENERAS LAS 30 PREGUNTAS ÚNICAS
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
      
      errorDiv.style.display = 'none';
      loadingDiv.style.display = 'flex';

      try {
        // Consultar estudiante con campos admin e institucion_id
        const { data, error } = await supabaseClient
          .from('estudiantes')
          .select('documento, nombre, admin, institucion_id')
          .eq('documento', documento)
          .single();

        loadingDiv.style.display = 'none';

        if (error || !data) {
          errorDiv.style.display = 'flex';
          return;
        }

        // Guardar datos del estudiante
        estudianteActual.documento = data.documento;
        estudianteActual.nombre = data.nombre;
        estudianteActual.esAdmin = data.admin || false;
        estudianteActual.institucionId = data.institucion_id;

        // Ocultar modal de autenticación
        document.getElementById('modal-autenticacion').style.display = 'none';

        // Si es admin, mostrar modal de administrador
        if (estudianteActual.esAdmin) {
          await mostrarModalAdmin();
        } else {
          // Si es estudiante regular, continuar normal
          mostrarInstrucciones();
        }

      } catch (error) {
        console.error('Error al consultar Supabase:', error);
        loadingDiv.style.display = 'none';
        errorDiv.innerHTML = '<span>❌</span><span>Error de conexión.</span>';
        errorDiv.style.display = 'flex';
      }
    });

    // ============================================
    // FUNCIONES ADMIN
    // ============================================
    async function mostrarModalAdmin() {
      console.log('🔐 Usuario administrador detectado');
      
      // Cargar instituciones activas
      const { data: instituciones, error } = await supabaseClient
        .from('instituciones')
        .select('id, nombre')
        .eq('activo', true)
        .order('nombre');
      
      if (error) {
        console.error('Error al cargar instituciones:', error);
        alert('Error al cargar instituciones');
        return;
      }
      
      // Llenar select
      const select = document.getElementById('institucion-select');
      select.innerHTML = '<option value="">-- Selecciona una institución --</option>';
      
      instituciones.forEach(inst => {
        const option = document.createElement('option');
        option.value = inst.id;
        option.textContent = inst.nombre;
        select.appendChild(option);
      });
      
      // Generar versión automática
      const fecha = new Date();
      const version = `v${fecha.getFullYear()}-${String(fecha.getMonth() + 1).padStart(2, '0')}`;
      document.getElementById('version-simulacro').value = version;
      
      // Mostrar modal
      document.getElementById('modal-admin').style.display = 'flex';
    }

    async function guardarPreguntasAdmin() {
      const institucionId = document.getElementById('institucion-select').value;
      const versionSimulacro = document.getElementById('version-simulacro').value.trim();
      
      if (!institucionId) {
        alert('Por favor selecciona una institución');
        return;
      }
      
      if (!versionSimulacro) {
        alert('Por favor ingresa la versión del simulacro');
        return;
      }
      
      console.log('💾 Guardando preguntas en base de datos...');
      
      // Preparar datos
      const preguntasParaGuardar = bancoPreguntas.map(p => ({
        institucion_id: institucionId,
        pregunta: p.pregunta,
        tema: p.tema,
        componente: p.componente,
        competencia: p.competencia,
        version_simulacro: versionSimulacro
      }));
      
      try {
        const { data, error } = await supabaseClient
          .from('preguntas_usadas')
          .insert(preguntasParaGuardar);
        
        if (error) throw error;
        
        console.log('✅ Preguntas guardadas exitosamente');
        alert(`✅ ${bancoPreguntas.length} preguntas guardadas.\n\nInstitución: ${document.getElementById('institucion-select').selectedOptions[0].text}\nVersión: ${versionSimulacro}`);
        
        // Cerrar modal y continuar
        document.getElementById('modal-admin').style.display = 'none';
        mostrarInstrucciones();
        
      } catch (error) {
        console.error('❌ Error al guardar preguntas:', error);
        alert('Error al guardar preguntas:\n\n' + error.message);
      }
    }

    function continuarSinGuardar() {
      console.log('⚠️ Continuando sin guardar preguntas');
      document.getElementById('modal-admin').style.display = 'none';
      mostrarInstrucciones();
    }

    function mostrarInstrucciones() {
      document.getElementById('nombre-estudiante').textContent = estudianteActual.nombre;
      document.getElementById('instrucciones').style.display = 'flex';
    }

    // ============================================
    // INICIAR TEST
    // ============================================
    document.getElementById('btn-comenzar').addEventListener('click', () => {
      estadoTest.horaInicio = new Date();
      
      document.getElementById('header-nombre').textContent = estudianteActual.nombre;
      document.getElementById('header-documento').textContent = estudianteActual.documento;

      document.getElementById('instrucciones').style.display = 'none';
      document.getElementById('contenedor-test').style.display = 'block';

      generarMapaPreguntas();
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
      // Verificar que TODAS las preguntas estén respondidas
      const respondidas = Object.keys(estadoTest.respuestas).length;
      const totalPreguntas = bancoPreguntas.length;
      
      if (respondidas < totalPreguntas) {
        const faltantes = totalPreguntas - respondidas;
        alert(`⚠️ Debes responder TODAS las preguntas.\n\nFaltan ${faltantes} pregunta(s) por responder.`);
        return;
      }

      // Preparar respuestas
      const objetoRespuestas = {};
      const respuestasDetalladas = {};
      
      bancoPreguntas.forEach(pregunta => {
        const respuestaEstudiante = estadoTest.respuestas[pregunta.numero];
        const esCorrecta = respuestaEstudiante === pregunta.respuesta_correcta;

        objetoRespuestas[pregunta.numero] = respuestaEstudiante;
        
        respuestasDetalladas[pregunta.numero] = {
          respuesta: respuestaEstudiante,
          competencia: pregunta.competencia,
          componente: pregunta.componente,
          tema: pregunta.tema,
          acierto: esCorrecta,
          respuesta_correcta: pregunta.respuesta_correcta
        };
      });

      // Preparar registro
      const registro = {
        documento_estudiante: estudianteActual.documento
        respuestas: objetoRespuestas,
        respuestas_detalladas: respuestasDetalladas
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
        alert('❌ Error al enviar respuestas.\n\n' + error.message);
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

---

## Flujo de Trabajo Completo

### Para Generar un Nuevo Simulacro:

1. **Consultar preguntas ya usadas** (manual, desde Supabase):
```sql
SELECT pregunta, tema, componente, competencia, version_simulacro
FROM preguntas_usadas
WHERE institucion_id = 'UUID_INSTITUCION'
ORDER BY created_at DESC;
```

2. **Formatear y proporcionar a Claude**:
```
Genera un simulacro con 30 preguntas NUEVAS.

⚠️ PREGUNTAS YA USADAS (NO REPETIR):

1. TEMA: Álgebra - Ecuaciones lineales
   PREGUNTA: Si 3x + 5 = 20...
   
2. TEMA: Geometría - Áreas
   PREGUNTA: Un rectángulo tiene...

[... listar todas las preguntas usadas ...]

Genera preguntas DIFERENTES sobre los mismos temas.
```

3. **Claude genera el artefacto** con:
    - 30 preguntas nuevas
    - Modal de admin incluido
    - Código para guardar preguntas en Supabase

4. **Usar el artefacto**:
    - Como admin: Te autenticas → Seleccionas institución → Guardas preguntas
    - Como estudiante: Te autenticas → Haces el simulacro normalmente

---

## Validaciones Críticas

### OBLIGATORIO: Todas las Preguntas Deben Estar Respondidas
```javascript
// Antes de enviar a Supabase
async function enviarRespuestas() {
  const respondidas = Object.keys(estadoTest.respuestas).length;
  const totalPreguntas = bancoPreguntas.length;
  
  if (respondidas < totalPreguntas) {
    const faltantes = totalPreguntas - respondidas;
    alert(`⚠️ Debes responder TODAS las preguntas.\n\nFaltan ${faltantes} pregunta(s) por responder.`);
    return; // NO enviar
  }
  
  // Continuar con el envío solo si todas están respondidas
  // ...
}
```

---

## Análisis de Resultados

### Análisis Individual (ya implementado)

Genera un análisis exhaustivo con:
- Puntaje global
- Desempeño por competencia
- Desempeño por componente
- Temas débiles específicos
- Plan de mejora personalizado

### Análisis Grupal (cuando se solicite)

Cuando se proporcionen consolidados de múltiples estudiantes de una institución:
```
Analiza estos resultados grupales:

INSTITUCIÓN: Colegio San José
TOTAL ESTUDIANTES: 35
DATOS: [JSON con respuestas de todos]

Genera informe que incluya:

1. ESTADÍSTICAS GENERALES
   - Promedio grupal
   - Distribución por niveles de desempeño
   - Mejor y peor puntaje

2. FORTALEZAS GRUPALES
   - Competencias donde sobresalen
   - Componentes con buen desempeño
   - Temas dominados
   - Qué están haciendo bien

3. DEBILIDADES GRUPALES
   - Competencias críticas
   - Componentes problemáticos
   - Temas con mayor dificultad
   - Preguntas más falladas

4. ANÁLISIS DE PATRONES
   - Errores comunes recurrentes
   - Correlaciones entre debilidades

5. RECOMENDACIONES PEDAGÓGICAS
   - Estrategias de enseñanza sugeridas
   - Temas prioritarios para reforzar
   - Recursos didácticos recomendados
   - Plan de intervención grupal

TONO: Constructivo, reconociendo fortalezas y señalando áreas de mejora.
```

---

## Restricciones

❌ NO usar localStorage/sessionStorage  
❌ NO importar imágenes externas  
❌ NO copiar preguntas literales de pruebas oficiales  
✅ USAR librería oficial de Supabase  
✅ GENERAR gráficos con código  
✅ INNOVAR sobre patrones ICFES  
✅ INCLUIR modal de administrador  
✅ VALIDAR respuesta completa antes de enviar

---

## Configuración de Supabase

En el código HTML, indica claramente:
```javascript
// ⚠️ CONFIGURAR AQUÍ TUS CREDENCIALES DE SUPABASE
const SUPABASE_URL = 'https://tu-proyecto.supabase.co';
const SUPABASE_ANON_KEY = 'tu-anon-key-aqui';
```

---

## Entregables

1. **Artefacto HTML funcional** con:
    - Modal de autenticación
    - Modal de administrador (NUEVO)
    - 30 preguntas únicas (10+ con gráficos)
    - Navegación completa
    - Validación de respuestas completas
    - Envío a Supabase con institucion_id

2. **Preguntas innovadoras** que:
    - NO repliquen las proporcionadas como "ya usadas"
    - Evalúen los mismos temas con enfoques diferentes
    - Mantengan calidad y nivel ICFES

---

¿Tienes los materiales oficiales ICFES y el listado de preguntas ya usadas (si aplica)? Una vez los tengas, genera el artefacto completo. 🚀