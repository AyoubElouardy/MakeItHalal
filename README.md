<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <title>MakeItHalal · app mejorada</title>
  <!-- Font Awesome Icons -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css" />
  <!-- Google Font -->
  <link href="https://fonts.googleapis.com/css2?family=Inter:opsz,wght@14..32,400;14..32,500;14..32,600;14..32,700&display=swap" rel="stylesheet" />
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }

    body {
      font-family: 'Inter', sans-serif;
      background: #ecf2f7;
      min-height: 100vh;
      display: flex;
      justify-content: center;
      align-items: center;
      padding: 16px;
    }

    .app-card {
      max-width: 500px;
      width: 100%;
      background: #ffffff;
      border-radius: 44px;
      box-shadow: 0 24px 60px rgba(0, 30, 40, 0.10);
      overflow: hidden;
      padding: 24px 22px 28px;
      transition: all 0.2s ease;
    }

    /* Header */
    .header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-bottom: 6px;
    }
    .logo {
      display: flex;
      align-items: center;
      gap: 10px;
    }
    .logo i {
      font-size: 32px;
      color: #1b7a42;
      background: #e2f0e8;
      padding: 8px;
      border-radius: 18px;
      box-shadow: 0 2px 6px rgba(0,0,0,0.02);
    }
    .logo h1 {
      font-size: 26px;
      font-weight: 700;
      letter-spacing: -0.4px;
      color: #0b2a35;
    }
    .badge-version {
      background: #dce8f0;
      padding: 4px 12px;
      border-radius: 40px;
      font-size: 12px;
      font-weight: 600;
      color: #1d4b5e;
    }

    .subhead {
      font-size: 14px;
      color: #3c5c6b;
      background: #f0f6fa;
      padding: 10px 16px;
      border-radius: 40px;
      margin: 10px 0 18px;
      display: inline-block;
      border-left: 4px solid #1f8b4c;
    }
    .subhead i {
      margin-right: 8px;
      color: #1f8b4c;
    }

    /* Cámara */
    .camera-wrapper {
      background: #e4ecf3;
      border-radius: 28px;
      overflow: hidden;
      margin-bottom: 16px;
      box-shadow: inset 0 2px 8px rgba(0,0,0,0.02);
    }

    #video {
      width: 100%;
      display: block;
      background: #1e2a32;
      min-height: 220px;
      object-fit: cover;
    }

    #canvas {
      display: none;
    }

    .capture-zone {
      display: flex;
      justify-content: center;
      gap: 12px;
      padding: 16px 16px 18px;
      background: #f7fafd;
      border-top: 1px solid #d6e2ec;
      flex-wrap: wrap;
    }

    .btn {
      border: none;
      background: white;
      padding: 12px 22px;
      border-radius: 60px;
      font-weight: 600;
      font-size: 15px;
      display: inline-flex;
      align-items: center;
      gap: 10px;
      transition: 0.12s ease;
      cursor: pointer;
      border: 1px solid #c6d5e2;
      color: #1d3a47;
      box-shadow: 0 2px 4px rgba(0,0,0,0.02);
    }
    .btn:active { transform: scale(0.96); }

    .btn-primary {
      background: #1b7a42;
      border: 1px solid #1b7a42;
      color: white;
      box-shadow: 0 8px 18px rgba(27, 122, 66, 0.18);
    }
    .btn-primary i { color: white; }
    .btn-primary:active { background: #146435; }

    .btn-secondary {
      background: #eef4f9;
      border-color: #cbdbe8;
    }
    .btn-secondary:active { background: #dfe8f0; }

    .btn:disabled {
      opacity: 0.5;
      pointer-events: none;
      filter: grayscale(0.3);
    }

    /* Panel de resultado */
    .result-panel {
      background: #f2f8fd;
      border-radius: 28px;
      padding: 20px 18px;
      margin: 12px 0 8px;
      border: 1px solid #dce8f2;
    }

    .status-row {
      display: flex;
      flex-wrap: wrap;
      align-items: center;
      justify-content: space-between;
      gap: 10px;
    }

    .badge-status {
      display: inline-flex;
      align-items: center;
      gap: 8px;
      font-weight: 700;
      font-size: 20px;
      padding: 6px 20px 6px 16px;
      border-radius: 60px;
      background: #b0c4d4;
      color: #1c2f3b;
      box-shadow: 0 2px 8px rgba(0,0,0,0.02);
    }
    .badge-halal {
      background: #1b7a42;
      color: white;
      box-shadow: 0 4px 12px rgba(27, 122, 66, 0.2);
    }
    .badge-haram {
      background: #b13e2b;
      color: white;
      box-shadow: 0 4px 12px rgba(177, 62, 43, 0.2);
    }
    .badge-unknown {
      background: #7e93a3;
      color: white;
    }

    .calories-box {
      background: white;
      border-radius: 60px;
      padding: 6px 18px 6px 14px;
      display: inline-flex;
      align-items: center;
      gap: 8px;
      font-weight: 600;
      font-size: 18px;
      border: 1px solid #c1d3e2;
      color: #10313e;
    }
    .calories-box i {
      color: #d48c2c;
      font-size: 18px;
    }

    .ingredient-section {
      margin-top: 16px;
    }
    .ingredient-label {
      font-size: 13px;
      font-weight: 600;
      color: #3b6277;
      letter-spacing: 0.3px;
      margin-bottom: 6px;
      display: flex;
      align-items: center;
      gap: 8px;
    }
    .ingredient-textarea {
      width: 100%;
      border: 1px solid #cbdae7;
      border-radius: 18px;
      padding: 14px 16px;
      font-family: 'Inter', sans-serif;
      font-size: 15px;
      line-height: 1.5;
      color: #10313e;
      background: white;
      resize: vertical;
      min-height: 72px;
      transition: 0.1s ease;
      box-shadow: inset 0 1px 4px rgba(0,0,0,0.02);
    }
    .ingredient-textarea:focus {
      outline: 2px solid #1b7a42;
      outline-offset: 2px;
      border-color: transparent;
    }

    .analysis-detail {
      font-size: 14px;
      color: #1d4a5e;
      background: #e4f0f8;
      padding: 10px 14px;
      border-radius: 18px;
      margin-top: 12px;
      border-left: 4px solid #1b7a42;
    }
    .analysis-detail i {
      margin-right: 8px;
      color: #1f6d8f;
    }

    .footnote {
      font-size: 13px;
      color: #4d6d7d;
      text-align: center;
      margin-top: 18px;
      border-top: 1px dashed #cbdbe8;
      padding-top: 16px;
      display: flex;
      justify-content: center;
      gap: 24px;
      flex-wrap: wrap;
    }
    .footnote i {
      margin-right: 4px;
      color: #2a6f8c;
    }

    @media (max-width: 440px) {
      .app-card { padding: 16px; }
      .btn { padding: 10px 16px; font-size: 14px; }
      .badge-status { font-size: 17px; padding: 4px 16px; }
    }
  </style>
</head>
<body>

<div class="app-card">
  <!-- Header -->
  <div class="header">
    <div class="logo">
      <i class="fas fa-seedling"></i>
      <h1>MakeItHalal</h1>
    </div>
    <span class="badge-version"><i class="fas fa-code-branch"></i> v2.0</span>
  </div>
  <div class="subhead">
    <i class="fas fa-camera-retro"></i> Captura los ingredientes · Edita si es necesario
  </div>

  <!-- Cámara -->
  <div class="camera-wrapper">
    <video id="video" autoplay playsinline muted></video>
    <canvas id="canvas"></canvas>
    <div class="capture-zone">
      <button class="btn btn-primary" id="captureBtn"><i class="fas fa-camera"></i> Capturar</button>
      <button class="btn btn-secondary" id="resetBtn"><i class="fas fa-undo-alt"></i> Reiniciar</button>
      <button class="btn btn-secondary" id="analyzeBtn"><i class="fas fa-microchip"></i> Analizar</button>
    </div>
  </div>

  <!-- Panel de resultado -->
  <div class="result-panel" id="resultPanel">
    <div class="status-row">
      <span id="statusBadge" class="badge-status badge-unknown">
        <i class="fas fa-circle"></i> Esperando...
      </span>
      <span class="calories-box" id="caloriesDisplay">
        <i class="fas fa-fire"></i> <span id="caloriesValue">--</span> kcal
      </span>
    </div>

    <div class="ingredient-section">
      <div class="ingredient-label">
        <i class="fas fa-list-ul"></i> Ingredientes detectados (editable)
      </div>
      <textarea id="ingredientTextarea" class="ingredient-textarea" placeholder="Los ingredientes aparecerán aquí. Puedes editarlos y presionar 'Analizar'.">Enfoca la lista de ingredientes y presiona Capturar.</textarea>
    </div>

    <div id="analysisDetail" class="analysis-detail">
      <i class="fas fa-info-circle"></i> <span id="detailMessage">Detalles del análisis aparecerán aquí.</span>
    </div>
  </div>

  <div class="footnote">
    <span><i class="fas fa-check-circle" style="color:#1b7a42;"></i> Halal: sin alcohol, cerdo ni derivados</span>
    <span><i class="fas fa-edit"></i> Edita el texto para mayor precisión</span>
  </div>
</div>

<script>
  (function() {
    "use strict";

    // Elementos
    const video = document.getElementById('video');
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');

    const captureBtn = document.getElementById('captureBtn');
    const resetBtn = document.getElementById('resetBtn');
    const analyzeBtn = document.getElementById('analyzeBtn');

    const statusBadge = document.getElementById('statusBadge');
    const caloriesValue = document.getElementById('caloriesValue');
    const ingredientTextarea = document.getElementById('ingredientTextarea');
    const detailMessage = document.getElementById('detailMessage');

    let stream = null;
    let isCameraReady = false;
    let currentDetectedText = '';

    // ----- Inicializar cámara -----
    async function initCamera() {
      try {
        if (stream) {
          stream.getTracks().forEach(track => track.stop());
        }
        const constraints = {
          video: { facingMode: 'environment', width: { ideal: 640 }, height: { ideal: 480 } }
        };
        stream = await navigator.mediaDevices.getUserMedia(constraints);
        video.srcObject = stream;
        await video.play();
        isCameraReady = true;
        captureBtn.disabled = false;
        canvas.width = video.videoWidth || 640;
        canvas.height = video.videoHeight || 480;
        setStatus('unknown', '📷 Listo');
        ingredientTextarea.value = 'Enfoca la lista de ingredientes y presiona Capturar.';
        caloriesValue.innerText = '--';
        detailMessage.innerText = 'Esperando captura...';
      } catch (err) {
        console.warn('Error cámara:', err);
        isCameraReady = false;
        captureBtn.disabled = true;
        setStatus('unknown', '⚠️ Sin cámara');
        ingredientTextarea.value = 'Permite el acceso a la cámara.';
        detailMessage.innerText = 'No se pudo acceder a la cámara.';
        alert('No se pudo acceder a la cámara. Verifica los permisos.');
      }
    }

    // ----- Capturar frame y simular OCR -----
    function captureAndSimulateOCR() {
      if (!isCameraReady || !video.videoWidth) {
        alert('La cámara no está lista.');
        return;
      }

      // Dibujar en canvas
      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;
      ctx.drawImage(video, 0, 0, canvas.width, canvas.height);

      // Simular OCR: seleccionar un texto aleatorio de una lista ampliada
      const mockIngredients = [
        "Harina de trigo, azúcar, aceite vegetal, sal, lecitina de soja, aroma natural de vainilla",
        "Agua, jarabe de glucosa, gelatina bovina, ácido cítrico, colorante carmín, sabor a fresa",
        "Leche entera, cacao en polvo, manteca de cacao, emulgente: lecitina, vainillina, azúcar",
        "Carne de res, almidón de maíz, especias, extracto de levadura, conservador sorbato potásico",
        "Agua carbonatada, azúcar, colorante caramelo, cafeína, ácido fosfórico, aroma natural",
        "Harina de arroz, almidón de patata, dextrosa, estabilizante goma xantana, sal",
        "Queso pasteurizado, crema, sal, enzimas, cultivos lácticos, cloruro de calcio",
        "Salsa de soja (agua, soja, trigo, sal), vinagre, ajo, jengibre, azúcar",
        "Cerveza (agua, malta de cebada, lúpulo), jarabe de maíz, extracto de malta",
        "Vino tinto, azúcar, especias, conservador sulfito, antioxidante",
        "Pechuga de pollo, agua, sal, especias, almidón modificado, proteína de soja",
        "Atún claro, aceite de oliva, sal, caldo de pescado, espesante",
        "Crema de cacahuete, aceite de palma, azúcar, sal, estabilizante",
        "Galleta: harina de trigo, aceite vegetal, azúcar, jarabe de glucosa, gasificante",
        "Bebida de avena, agua, avena, aceite de girasol, sal, vitaminas"
      ];
      const randomIdx = Math.floor(Math.random() * mockIngredients.length);
      const detectedText = mockIngredients[randomIdx];
      currentDetectedText = detectedText;

      // Mostrar en el textarea
      ingredientTextarea.value = detectedText;
      detailMessage.innerText = 'Texto extraído (simulación OCR). Revisa y edita si es necesario.';

      // Análisis automático (para vista previa)
      performAnalysis(detectedText);
    }

    // ----- Análisis principal (recibe texto) -----
    function performAnalysis(text) {
      if (!text || text.trim() === '') {
        setStatus('unknown', '⚠️ Sin texto');
        caloriesValue.innerText = '--';
        detailMessage.innerText = 'No hay ingredientes para analizar.';
        return;
      }

      const lower = text.toLowerCase();
      // Palabras clave ampliadas
      const haramKeywords = [
        'cerdo', 'pork', 'gelatina de cerdo', 'gelatin pork', 'cerveza', 'vino', 'alcohol', 'sidra', 'licor',
        'brandy', 'coñac', 'vodka', 'whisky', 'ron', 'ginebra', 'bebida alcohólica', 'sulfito', 'carmín',
        'cochineal', 'gelatina bovina', 'manteca de cerdo', 'estearina de cerdo', 'grasa de cerdo',
        'emulgente e471', 'e471', 'e120', 'e904', 'e1105', 'jugo de cerdo', 'tocino', 'jamón', 'chorizo',
        'salami', 'pepperoni', 'lardo', 'panceta', 'gelatina de puerco'
      ];
      const questionableKeywords = ['gelatina', 'gelatin', 'e441', 'estearina', 'glicerina', 'glicerol', 'emulgente'];

      let isHalal = true;
      let reason = '';

      // Detectar Haram
      for (let kw of haramKeywords) {
        if (lower.includes(kw)) {
          isHalal = false;
          reason = `contiene "${kw}"`;
          break;
        }
      }

      // Si no es Haram, revisar cuestionable
      if (isHalal) {
        for (let qw of questionableKeywords) {
          if (lower.includes(qw)) {
            // Si contiene "gelatina" pero no especifica bovina/cerdo, es dudoso
            if (qw === 'gelatina' || qw === 'gelatin') {
              if (!lower.includes('bovina') && !lower.includes('cerdo') && !lower.includes('pork')) {
                isHalal = null;
                reason = `"${qw}" sin origen especificado`;
                break;
              }
            } else {
              isHalal = null;
              reason = `contiene "${qw}" (puede ser dudoso)`;
              break;
            }
          }
        }
      }

      // Actualizar badge
      if (isHalal === true) {
        setStatus('halal', '✅ Halal');
        detailMessage.innerText = `✅ Producto permitido. ${reason ? 'Nota: ' + reason : 'Sin ingredientes prohibidos.'}`;
      } else if (isHalal === false) {
        setStatus('haram', '❌ No Halal (Haram)');
        detailMessage.innerText = `❌ No apto. Motivo: ${reason}`;
      } else {
        setStatus('unknown', '⚠️ Dudoso / sin certeza');
        detailMessage.innerText = `⚠️ Ingrediente cuestionable: ${reason}. Recomendamos consultar a una autoridad.`;
      }

      // Calcular calorías (mejorado)
      let cal = 80;
      if (lower.includes('aceite') || lower.includes('manteca') || lower.includes('crema') || lower.includes('palma')) cal += 70;
      if (lower.includes('azúcar') || lower.includes('jarabe') || lower.includes('glucosa') || lower.includes('miel')) cal += 55;
      if (lower.includes('carne') || lower.includes('res') || lower.includes('pollo') || lower.includes('cordero')) cal += 60;
      if (lower.includes('queso') || lower.includes('leche') || lower.includes('yogur')) cal += 45;
      if (lower.includes('cacao') || lower.includes('chocolate')) cal += 35;
      if (lower.includes('fruta') || lower.includes('manzana') || lower.includes('plátano')) cal += 20;
      if (lower.includes('pescado') || lower.includes('atún') || lower.includes('salmón')) cal += 40;
      if (lower.includes('avena') || lower.includes('cereal') || lower.includes('trigo')) cal += 30;
      // Factor aleatorio para variar
      cal += Math.floor(Math.random() * 25) - 5;
      cal = Math.max(35, Math.min(500, cal));
      caloriesValue.innerText = cal;
    }

    // ----- setStatus (actualiza badge) -----
    function setStatus(type, label) {
      statusBadge.className = 'badge-status';
      if (type === 'halal') {
        statusBadge.classList.add('badge-halal');
        statusBadge.innerHTML = `<i class="fas fa-check-circle"></i> ${label}`;
      } else if (type === 'haram') {
        statusBadge.classList.add('badge-haram');
        statusBadge.innerHTML = `<i class="fas fa-times-circle"></i> ${label}`;
      } else {
        statusBadge.classList.add('badge-unknown');
        statusBadge.innerHTML = `<i class="fas fa-question-circle"></i> ${label}`;
      }
    }

    // ----- Reset -----
    function resetApp() {
      ingredientTextarea.value = 'Enfoca la lista de ingredientes y presiona Capturar.';
      caloriesValue.innerText = '--';
      setStatus('unknown', '📷 Listo');
      detailMessage.innerText = 'Esperando captura o análisis.';
      currentDetectedText = '';
      ctx.clearRect(0, 0, canvas.width, canvas.height);
    }

    // ----- Eventos -----
    captureBtn.addEventListener('click', captureAndSimulateOCR);
    resetBtn.addEventListener('click', resetApp);
    analyzeBtn.addEventListener('click', function() {
      const text = ingredientTextarea.value.trim();
      if (!text) {
        alert('No hay texto para analizar. Escribe o captura los ingredientes.');
        return;
      }
      performAnalysis(text);
    });

    // Iniciar cámara
    initCamera();

    // Ajustar canvas cuando el video cargue
    video.addEventListener('loadedmetadata', () => {
      if (video.videoWidth) {
        canvas.width = video.videoWidth;
        canvas.height = video.videoHeight;
      }
    });

    console.log('🍃 MakeItHalal v2.0 · mejorada con edición de ingredientes');
  })();
</script>
</body>
</html>
