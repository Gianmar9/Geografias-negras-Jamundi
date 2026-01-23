<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Interacciones con imágenes</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    html, body {
      height: 100%;
      overflow: hidden;
      background: #000;
      font-family: system-ui, sans-serif;
    }

    main {
      position: relative;
      width: 100vw;
      height: 100vh;
      overflow: hidden;
    }

    /* CADA PANTALLA UNA ENCIMA DE OTRA, CON FADE */
    .screen {
      position: absolute;
      inset: 0;
      width: 100%;
      height: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
      overflow: hidden;

      opacity: 0;
      pointer-events: none;
      transition: opacity 0.6s ease;
    }

    .screen.active {
      opacity: 1;
      pointer-events: auto;
      z-index: 1;
    }

    .bg-image {
      position: absolute;
      inset: 0;
      width: 100%;
      height: 100%;
      object-fit: cover;
      z-index: 0;
    }

    /* 🔧 UBICACIÓN DEL BOTÓN PLAY */
    .play-button {
      cursor: pointer;
      width: 200px;
      max-width: 50vmin;
      transition: transform 0.25s ease, filter 0.25s ease;
      position: absolute;
      z-index: 1;

      top: 70%;
      left: 50%;
      transform: translate(-50%, -50%);
    }

    .play-button:hover {
      transform: translate(-50%, -50%) scale(1.08);
      filter: drop-shadow(0 0 12px rgba(255, 255, 255, 0.7));
    }

    /* CONTENEDOR GENERAL DE BOTONES INTRODUCCIÓN */
    .intro-buttons {
      position: absolute;
      inset: 0;
      z-index: 1;
      pointer-events: none;
    }

    /* GRUPO VERTICAL: FINCA, HUMEDALES, VIVIENDA */
    .buttons-column {
      position: absolute;
      display: flex;
      flex-direction: column;
      gap: 1.5rem;
      pointer-events: auto;

      top: 5%;
      left: 4%;
    }

    /* BOTÓN TALLERES INDEPENDIENTE */
    .talleres-wrapper {
      position: absolute;
      pointer-events: auto;

      top: 85%;
      left: 80%;
    }

    .btn-wrapper {
      position: relative;
      display: inline-block;
    }

    .image-button {
      cursor: pointer;
      max-width: 950px;
      width: 35vmin;
      transition: transform 0.25s ease, filter 0.25s ease;
      position: relative;
      z-index: 1;
    }

    .image-button:hover {
      transform: scale(1.05);
      filter: drop-shadow(0 0 8px rgba(255, 255, 255, 0.6));
    }

    /* VENTANA EMERGENTE GENERAL (3 BOTONES) */
    .preview {
      position: absolute;
      bottom: -430%;
      left: 110%;
      transform: translateX(-50%);
      opacity: 0;
      pointer-events: none;
      transition: opacity 0.25s ease, transform 0.25s ease;
      max-width: 850px;
      width: 150vmin;
      border-radius: 8px;
      border: 2px solid rgba(255, 255, 255, 0.8);
      box-shadow: 0 10px 25px rgba(0, 0, 0, 0.7);
      background: #000;
      z-index: 2;
    }

    /* 🔧 VENTANA EMERGENTE INDEPENDIENTE PARA TALLERES */
    .preview-talleres {
      position: absolute;
      bottom: 0%;
      left: -265%;
      transform: translateX(0);
      opacity: 0;
      pointer-events: none;
      transition: opacity 0.25s ease, transform 0.25s ease;
      max-width: 850px;
      width: 150vmin;
      border-radius: 8px;
      border: 2px solid rgba(255, 255, 255, 0.8);
      box-shadow: 0 10px 25px rgba(0, 0, 0, 0.7);
      background: #000;
      z-index: 3;
    }

    .btn-wrapper:hover .preview,
    .btn-wrapper:hover .preview-talleres {
      opacity: 1;
      transform: translateY(-4px);
    }

    .back-btn {
      position: absolute;
      top: 1.5rem;
      left: 1.5rem;
      padding: 0.6rem 1rem;
      border-radius: 999px;
      border: 1px solid rgba(255, 255, 255, 0.7);
      background: rgba(0, 0, 0, 0.55);
      color: #fff;
      font-size: 0.9rem;
      cursor: pointer;
      backdrop-filter: blur(8px);
      transition: background 0.2s ease, transform 0.2s ease;
      z-index: 20;
    }

    .back-btn:hover {
      background: rgba(255, 255, 255, 0.1);
      transform: translateY(-1px);
    }

    /* ⭐ CAPA DE PUNTOS PARPADEANDO EN LA SECCIÓN FINCA */
    .puntos-fincas {
      position: absolute;
      inset: 0;
      width: 100%;
      height: 100%;
      object-fit: cover;
      pointer-events: none;
      z-index: 1;
      animation: titilar 1.3s infinite ease-in-out;
    }

    @keyframes titilar {
      0%   { opacity: 0.2; }
      50%  { opacity: 1; }
      100% { opacity: 0.2; }
    }

    /* ⭐⭐ CAPAS PRENDER/APAGAR EN FINCA ⭐⭐ */
    .layer-image {
      position: absolute;
      inset: 0;
      width: 100%;
      height: 100%;
      object-fit: cover;
      pointer-events: none;
      opacity: 0;
      transition: opacity 0.25s ease;
      z-index: 2; /* Encima de puntosfincas, debajo de botones */
    }

    .layer-image.visible {
      opacity: 1;
    }

    /* PANEL DE BOTONES DE CAPAS EN FINCA */
    .layers-panel {
      position: absolute;
      top: 10%;
      right: 3%;
      display: flex;
      flex-direction: column;
      gap: 0.4rem;
      z-index: 30;
    }

    .layer-toggle-btn {
      pointer-events: auto;
      padding: 0.25rem 0.6rem;
      font-size: 0.7rem;
      border-radius: 999px;
      border: 1px solid rgba(255, 255, 255, 0.6);
      background: rgba(0, 0, 0, 0.6);
      color: #fff;
      cursor: pointer;
      text-align: left;
      white-space: nowrap;
      max-width: 220px;
      overflow: hidden;
      text-overflow: ellipsis;
      backdrop-filter: blur(6px);
      transition: background 0.2s ease, transform 0.2s ease;
    }

    .layer-toggle-btn:hover {
      background: rgba(255, 255, 255, 0.12);
      transform: translateY(-1px);
    }

    .layer-toggle-btn.active {
      background: rgba(144, 238, 144, 0.2);
      border-color: rgba(144, 238, 144, 0.9);
    }

    /* ⭐⭐ BOTONES DE CONSEJOS EN FINCA ⭐⭐ */
    .consejo-btn {
      position: absolute;
      pointer-events: auto;
      padding: 0.35rem 0.7rem;
      font-size: 0.7rem;
      border-radius: 999px;
      border: 1px solid rgba(255, 255, 255, 0.7);
      background: rgba(0, 0, 0, 0.7);
      color: #fff;
      cursor: pointer;
      white-space: nowrap;
      backdrop-filter: blur(6px);
      transition: background 0.2s ease, transform 0.2s ease;
      z-index: 35; /* por encima de las capas */
    }

    .consejo-btn:hover {
      background: rgba(255, 255, 255, 0.12);
      transform: translateY(-1px);
    }

    /* POSICIÓN DE CADA BOTÓN (AJUSTA LIBREMENTE top / left) */
    .consejo-bocas     { top: 22%; left: 66%; }
    .consejo-robles    { top: 72%; left: 28%; }
    .consejo-villapaz  { top: 75%; left: 48%; }
    .consejo-chagres   { top: 80%; left: 25%; }
    .consejo-quinamayo { top: 75%; left: 40%; }
    .consejo-laventura { top: 60%; left: 55%; }
    .consejo-paso      { top: 44%; left: 56%; }
    .consejo-sanisidro { top: 18%; left: 60%; }
    .consejo-varejonal { top: 58%; left: 28%; }

    /* BOTÓN PARA HUMEDALES: FINCA TRADICIONAL Y EROSIÓN DEL RÍO */
    .rio-finca-btn {
      position: absolute;
      top: 75%;
      left: 48%;
      transform: translate(-50%, -50%);
      padding: 0.5rem 1rem;
      font-size: 0.75rem;
      border-radius: 999px;
      border: 1px solid rgba(255, 255, 255, 0.8);
      background: rgba(0, 0, 0, 0.7);
      color: #fff;
      cursor: pointer;
      white-space: nowrap;
      backdrop-filter: blur(6px);
      z-index: 35;
      transition: background 0.2s ease, transform 0.2s ease;
    }

    .rio-finca-btn:hover {
      background: rgba(255, 255, 255, 0.12);
      transform: translate(-50%, -52%);
    }

    /* MODAL (COMPARTIDO POR DIFERENTES SECCIONES) */
    .modal-overlay {
      position: absolute;
      inset: 0;
      background: rgba(0, 0, 0, 0.75);
      display: flex;
      justify-content: center;
      align-items: center;
      opacity: 0;
      pointer-events: none;
      transition: opacity 0.25s ease;
      z-index: 60;
    }

    .modal-overlay.active {
      opacity: 1;
      pointer-events: auto;
    }

    .modal-content {
      position: relative;
      max-width: 80vw;
      max-height: 80vh;
      background: #000;
      border-radius: 10px;
      border: 2px solid rgba(255, 255, 255, 0.8);
      overflow: hidden;
      box-shadow: 0 12px 30px rgba(0, 0, 0, 0.8);
    }

    .modal-content img {
      display: block;
      max-width: 100%;
      max-height: 100%;
      object-fit: contain;
    }

    .modal-close {
      position: absolute;
      top: 0.4rem;
      right: 0.4rem;
      padding: 0.25rem 0.6rem;
      font-size: 0.7rem;
      border-radius: 999px;
      border: 1px solid rgba(255, 255, 255, 0.7);
      background: rgba(0, 0, 0, 0.7);
      color: #fff;
      cursor: pointer;
    }

    .modal-close:hover {
      background: rgba(255, 255, 255, 0.12);
    }
  </style>
</head>

<body>

<main>

  <!-- SECCIÓN: BIENVENIDA -->
  <section id="bienvenida" class="screen active">
    <img src="Bienvenida.png" alt="Bienvenida" class="bg-image">

    <img
      src="boton_play.png"
      alt="Play"
      class="play-button nav-btn"
      data-target="#introduccion"
    >
  </section>

  <!-- SECCIÓN: INTRODUCCIÓN -->
  <section id="introduccion" class="screen">
    <img src="introduccion.png" alt="Introducción" class="bg-image">

    <div class="intro-buttons">

      <!-- GRUPO DE 3 BOTONES -->
      <div class="buttons-column">

        <div class="btn-wrapper">
          <img src="Boton_finca.png" class="image-button nav-btn" data-target="#finca">
          <img src="finca_imagen.png" class="preview">
        </div>

        <div class="btn-wrapper">
          <img src="Boton_humedales.png" class="image-button nav-btn" data-target="#humedales">
          <img src="humedal_imagen.png" class="preview">
        </div>

        <div class="btn-wrapper">
          <img src="Boton_vivienda.png" class="image-button nav-btn" data-target="#vivienda">
          <img src="vivienda_imagen.png" class="preview">
        </div>

      </div>

      <!-- BOTÓN TALLERES INDEPENDIENTE -->
      <div class="btn-wrapper talleres-wrapper">
        <img src="Boton_talleres.png" class="image-button nav-btn" data-target="#talleres">
        <img src="talleres_imagen.png" class="preview-talleres">
      </div>

    </div>
  </section>

  <!-- SECCIONES INTERNAS -->
  <section id="finca" class="screen">
    <img src="finca_imagen.png" class="bg-image">

    <!-- CAPA DE PUNTOS PARPADEANDO -->
    <img src="puntosfincas.png" class="puntos-fincas" alt="Puntos de fincas">

    <!-- ⭐ CAPAS QUE SE PRENDEN Y APAGAN (ajusta los nombres de archivo) -->
    <img src="Cultivos_por_finca.png"
         class="layer-image"
         data-layer="cultivos">

    <img src="Plantas_medicinales_por_finca.png"
         class="layer-image"
         data-layer="plantas">

    <img src="Animales_por_finca.png"
         class="layer-image"
         data-layer="animales">

    <img src="Produccion_agricola_por_finca.png"
         class="layer-image"
         data-layer="prod_agricola">

    <img src="Area_de_las_fincas.png"
         class="layer-image"
         data-layer="area">

    <img src="Afectaciones_de_la_finca.png"
         class="layer-image"
         data-layer="afectaciones">

    <img src="Produccion_animal.png"
         class="layer-image"
         data-layer="prod_animal">

    <img src="Genero_del_dueno.png"
         class="layer-image"
         data-layer="genero">

    <img src="Pertenencia_consejo_comunitario.png"
         class="layer-image"
         data-layer="consejo">

    <!-- PANEL DE BOTONES PARA PRENDER/APAGAR -->
    <div class="layers-panel">
      <button class="layer-toggle-btn" data-layer-target="cultivos">
        1. Cultivos por finca
      </button>
      <button class="layer-toggle-btn" data-layer-target="plantas">
        2. Plantas medicinales por finca
      </button>
      <button class="layer-toggle-btn" data-layer-target="animales">
        3. Animales por finca
      </button>
      <button class="layer-toggle-btn" data-layer-target="prod_agricola">
        4. Producción agrícola por finca
      </button>
      <button class="layer-toggle-btn" data-layer-target="area">
        5. Área de las fincas
      </button>
      <button class="layer-toggle-btn" data-layer-target="afectaciones">
        6. Afectaciones de la finca
      </button>
      <button class="layer-toggle-btn" data-layer-target="prod_animal">
        7. Calidad del suelo
      </button>
      <button class="layer-toggle-btn" data-layer-target="genero">
        8. Género del dueñ@
      </button>
      <button class="layer-toggle-btn" data-layer-target="consejo">
        9. Pertenencia a consejo comunitario
      </button>
    </div>

    <!-- ⭐ BOTONES DE CONSEJOS (MÓVILES POR top/left) -->
    <button
      class="consejo-btn consejo-bocas"
      data-img="consejo1.png">
      Bocas del Palo
    </button>

    <button
      class="consejo-btn consejo-robles"
      data-img="consejo2.png">
      Robles
    </button>

    <button
      class="consejo-btn consejo-villapaz"
      data-img="consejo3.png">
      Villa Paz
    </button>

    <button
      class="consejo-btn consejo-chagres"
      data-img="consejo4.png">
      Chagres
    </button>

    <button
      class="consejo-btn consejo-quinamayo"
      data-img="consejo5.png">
      Quinamayo
    </button>

    <button
      class="consejo-btn consejo-laventura"
      data-img="consejo6.png">
      La Ventura
    </button>

    <button
      class="consejo-btn consejo-paso"
      data-img="consejo7.png">
      Paso de La Bolsa
    </button>

    <button
      class="consejo-btn consejo-sanisidro"
      data-img="consejo8.png">
      San Isidro
    </button>

    <button
      class="consejo-btn consejo-varejonal"
      data-img="consejo9.png">
      Varejonal
    </button>

    <!-- MODAL REUTILIZABLE PARA TODOS LOS CONSEJOS -->
    <div class="modal-overlay" id="consejo-modal">
      <div class="modal-content">
        <img id="consejo-modal-img" src="" alt="Consejo comunitario">
        <button class="modal-close" type="button">Cerrar</button>
      </div>
    </div>

    <button class="back-btn nav-btn" data-target="#introduccion">⬅ Volver</button>
  </section>

  <section id="humedales" class="screen">
    <img src="humedal_imagen.png" class="bg-image">

    <!-- BOTÓN: FINCA TRADICIONAL Y EROSIÓN DEL RÍO -->
    <button
      class="rio-finca-btn"
      type="button"
      id="btn-rio-finca">
      FINCA TRADICIONAL Y EROSIÓN DEL RÍO
    </button>

    <!-- MODAL PARA LA IMAGEN DE FINCA TRADICIONAL Y EROSIÓN DEL RÍO -->
    <div class="modal-overlay" id="rio-finca-modal">
      <div class="modal-content">
        <img src="finca_tradicional_erosion_rio.png" alt="Finca tradicional y erosión del río">
        <button class="modal-close" type="button">Cerrar</button>
      </div>
    </div>

    <button class="back-btn nav-btn" data-target="#introduccion">⬅ Volver</button>
  </section>

  <section id="vivienda" class="screen">
    <img src="vivienda_imagen.png" class="bg-image">
    <button class="back-btn nav-btn" data-target="#introduccion">⬅ Volver</button>
  </section>

  <section id="talleres" class="screen">
    <img src="talleres_imagen.png" class="bg-image">
    <button class="back-btn nav-btn" data-target="#introduccion">⬅ Volver</button>
  </section>

</main>

<script>
  const screens = document.querySelectorAll('.screen');

  function showScreen(targetSelector) {
    const target = document.querySelector(targetSelector);
    if (!target) return;

    screens.forEach(s => s.classList.remove('active'));
    target.classList.add('active');
  }

  // Navegación entre pantallas
  document.querySelectorAll('.nav-btn').forEach(btn => {
    btn.addEventListener('click', () => {
      const target = btn.dataset.target;
      if (!target) return;
      showScreen(target);
    });
  });

  // ⭐ LÓGICA PARA PRENDER / APAGAR CAPAS EN FINCA
  const layerButtons = document.querySelectorAll('.layer-toggle-btn');

  layerButtons.forEach(btn => {
    btn.addEventListener('click', () => {
      const layerName = btn.dataset.layerTarget;
      const layerImg = document.querySelector(
        '.layer-image[data-layer="' + layerName + '"]'
      );
      if (!layerImg) return;

      layerImg.classList.toggle('visible');
      btn.classList.toggle('active');
    });
  });

  // ⭐ LÓGICA DE MODAL PARA CONSEJOS
  const consejoButtons = document.querySelectorAll('.consejo-btn');
  const consejoModal = document.getElementById('consejo-modal');
  const consejoModalImg = document.getElementById('consejo-modal-img');
  const consejoModalClose = consejoModal.querySelector('.modal-close');

  consejoButtons.forEach(btn => {
    btn.addEventListener('click', () => {
      const imgSrc = btn.dataset.img;
      if (!imgSrc) return;
      consejoModalImg.src = imgSrc;
      consejoModal.classList.add('active');
    });
  });

  consejoModalClose.addEventListener('click', () => {
    consejoModal.classList.remove('active');
  });

  // Cerrar modal de consejos haciendo click fuera del contenido
  consejoModal.addEventListener('click', (e) => {
    if (e.target === consejoModal) {
      consejoModal.classList.remove('active');
    }
  });

  // ⭐ LÓGICA DE MODAL PARA "FINCA TRADICIONAL Y EROSIÓN DEL RÍO" EN HUMEDALES
  const rioFincaBtn = document.getElementById('btn-rio-finca');
  const rioFincaModal = document.getElementById('rio-finca-modal');
  const rioFincaClose = rioFincaModal.querySelector('.modal-close');

  rioFincaBtn.addEventListener('click', () => {
    rioFincaModal.classList.add('active');
  });

  rioFincaClose.addEventListener('click', () => {
    rioFincaModal.classList.remove('active');
  });

  rioFincaModal.addEventListener('click', (e) => {
    if (e.target === rioFincaModal) {
      rioFincaModal.classList.remove('active');
    }
  });
</script>

</body>
</html>
