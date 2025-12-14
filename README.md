
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Generador PDF de Identificación Profesional</title>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/1.6.1/cropper.min.css"/>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/1.6.1/cropper.min.js"></script>

    <style>
        /* 🎨 Variables de Diseño y Colores */
        :root {
            --primary: #4f46e5; /* Índigo */
            --secondary: #10b981; /* Esmeralda */
            --danger: #ef4444; /* Rojo */
            --bg-color: #f3f4f6; /* Gris claro para fondo */
            --card-color: #ffffff;
            --text-color: #1f2937; /* Gris oscuro */
            --shadow-light: 0 10px 20px rgba(0, 0, 0, 0.08);
            --shadow-heavy: 0 25px 50px rgba(0, 0, 0, 0.15);
        }

        /* 💫 Estilo del Fondo Animado (Opcional: puedes quitarlo si no quieres la animación) */
        @keyframes moveBackground {
            0% { background-position: 0% 50%; }
            50% { background-position: 100% 50%; }
            100% { background-position: 0% 50%; }
        }

        /* 📐 Estilos Base */
        * {
            box-sizing: border-box;
        }

        body {
            margin: 0;
            font-family: 'Inter', sans-serif;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            color: var(--text-color);
            background: linear-gradient(135deg, #dbeafe, #e0f2fe, #fff); /* Fondo gradiente */
            background-size: 400% 400%;
            animation: moveBackground 20s ease infinite; /* Animación de fondo */
            padding: 20px;
        }

        /* 📱 Contenedor Principal de la Aplicación */
        .app {
            width: 100%;
            max-width: 950px;
            background: var(--card-color);
            border-radius: 24px;
            box-shadow: var(--shadow-heavy);
            padding: 40px;
            text-align: center;
        }

        /* 📰 Encabezado */
        .header h1 {
            margin: 0 0 10px 0;
            font-size: 2.5rem;
            color: var(--primary);
            font-weight: 700;
        }

        .header p {
            color: #6b7280;
            margin-bottom: 30px;
        }

        /* 📸 Tarjetas para Cargar Imágenes */
        .cards {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
            gap: 25px;
            margin-bottom: 30px;
        }

        .card {
            background: linear-gradient(145deg, var(--card-color), #f9fafb);
            border-radius: 18px;
            padding: 30px;
            text-align: center;
            cursor: pointer;
            border: 2px solid var(--bg-color);
            transition: all 0.3s ease;
            box-shadow: var(--shadow-light);
        }

        .card:hover {
            transform: translateY(-5px) scale(1.02);
            border-color: var(--primary);
            box-shadow: 0 15px 30px rgba(79, 70, 229, 0.2);
        }

        .card span {
            font-size: 3.5rem;
            display: block;
            margin-bottom: 10px;
        }

        .card h3 {
            margin: 0;
            font-weight: 600;
            font-size: 1.5rem;
        }

        .card p {
            color: #9ca3af;
            font-size: 0.9rem;
            margin-top: 5px;
        }

        /* 🖼️ Previsualización */
        .preview {
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 20px;
            min-height: 150px; /* Para mantener espacio */
            margin-bottom: 30px;
        }

        .preview img {
            width: 100%;
            max-width: 45%;
            aspect-ratio: 1.586 / 1; /* Relación de aspecto estándar de ID */
            object-fit: cover;
            border-radius: 12px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
            border: 3px solid #e5e7eb;
            transition: transform 0.3s ease;
        }

        .preview img:hover {
            transform: scale(1.03);
        }

        .placeholder {
            color: #94a3b8;
            font-style: italic;
            padding: 30px;
            font-size: 1.1rem;
        }

        /* 📢 Estado */
        .status {
            font-weight: 700;
            margin-bottom: 25px;
            padding: 10px;
            border-radius: 8px;
            color: var(--text-color);
        }

        .status[data-complete="true"] {
            color: var(--secondary);
            background-color: #ecfdf5;
        }
        .status[data-complete="false"] {
            color: #f59e0b;
            background-color: #fffbeb;
        }

        /* ⬇️ Botón Principal */
        .generate-btn {
            width: 100%;
            padding: 18px;
            font-size: 1.3rem;
            font-weight: 700;
            background: linear-gradient(90deg, var(--secondary), #16a34a);
            border: none;
            color: white;
            border-radius: 14px;
            cursor: pointer;
            transition: all 0.3s ease;
            box-shadow: 0 10px 20px rgba(16, 185, 129, 0.3);
        }

        .generate-btn:hover:not(:disabled) {
            transform: translateY(-2px);
            box-shadow: 0 15px 25px rgba(16, 185, 129, 0.4);
        }

        .generate-btn:disabled {
            background: #94a3b8;
            box-shadow: none;
            cursor: not-allowed;
        }

        /* 🔳 MODALES */
        .modal {
            display: none;
            position: fixed;
            inset: 0;
            background: rgba(0, 0, 0, 0.6);
            backdrop-filter: blur(5px); /* Efecto de desenfoque */
            justify-content: center;
            align-items: center;
            z-index: 1000;
            padding: 20px;
        }

        .modal-box {
            background: var(--card-color);
            border-radius: 16px;
            padding: 30px;
            width: 90%;
            max-width: 450px;
            box-shadow: var(--shadow-heavy);
            transform: scale(0.95);
            opacity: 0;
            transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }
        
        .modal.active .modal-box {
            transform: scale(1);
            opacity: 1;
        }
        
        .modal-box button {
            width: 100%;
            padding: 16px;
            margin-top: 15px;
            border: none;
            border-radius: 12px;
            font-size: 1.1rem;
            font-weight: 600;
            cursor: pointer;
            transition: background-color 0.2s, transform 0.2s;
        }

        .camera { background: var(--primary); color: white; }
        .gallery { background: #0ea5e9; color: white; }
        .cancel { background: #e5e7eb; color: var(--text-color); }
        .rotate { background: #f97316; color: white; }
        .confirm { background: var(--secondary); color: white; }
        
        .modal-box button:hover:not(.cancel) {
            filter: brightness(1.1);
        }
        .modal-box button:active {
            transform: scale(0.98);
        }

        /* Estilos específicos para el modal de recorte */
        #cropModal .modal-box {
            max-width: 500px;
        }
        
        .crop-img {
            display: block;
            max-width: 100%;
            border-radius: 8px;
        }

    </style>
</head>

<body>

<div class="app">

    <div class="header">
        <h1>🪪 Generador PDF de Identificación</h1>
        <p>Sube, recorta y genera tu documento de identificación profesional en formato PDF.</p>
    </div>

    <div class="cards">
        <div class="card" onclick="openSelection('frente')">
            <span>📸</span>
            <h3>Frente de la Identificación</h3>
            <p>Toca para subir la imagen frontal.</p>
        </div>

        <div class="card" onclick="openSelection('reverso')">
            <span>📷</span>
            <h3>Reverso de la Identificación</h3>
            <p>Toca para subir la imagen trasera.</p>
        </div>
    </div>

    <input type="file" id="frente-camera" accept="image/*" capture hidden>
    <input type="file" id="frente-gallery" accept="image/*" hidden>
    <input type="file" id="reverso-camera" accept="image/*" capture hidden>
    <input type="file" id="reverso-gallery" accept="image/*" hidden>

    <div class="preview">
        <img id="prev-frente" style="display: none;">
        <img id="prev-reverso" style="display: none;">
        <div id="placeholder" class="placeholder">Aún no hay imágenes cargadas</div>
    </div>

    <div id="status" class="status" data-complete="false">📄 Falta subir imágenes</div>

    <button id="btnPDF" class="generate-btn" disabled>
        ⬇️ Generar PDF
    </button>

</div>

<div id="selectModal" class="modal">
    <div class="modal-box">
        <button class="camera" onclick="choose('camera')">📷 Abrir Cámara</button>
        <button class="gallery" onclick="choose('gallery')">🖼️ Seleccionar de Galería</button>
        <button class="cancel" onclick="closeSelect()">❌ Cancelar</button>
    </div>
</div>

<div id="cropModal" class="modal">
    <div class="modal-box">
        <div style="max-height: 70vh; overflow: auto; margin-bottom: 20px;">
            <img id="cropImg" class="crop-img">
        </div>
        <button class="rotate" onclick="rotateImg()">🔄 Rotar Imagen</button>
        <button class="confirm" onclick="confirmCrop()">✔️ Confirmar Recorte</button>
        <button class="cancel" onclick="closeCrop()">❌ Cancelar</button>
    </div>
</div>

<script>
    const { jsPDF } = window.jspdf;
    // Variables de estado
    let frente = null;
    let reverso = null;
    let side = null; // 'frente' o 'reverso'
    let cropper = null;
    let rotation = 0;

    // Elementos del DOM
    const prevF = document.getElementById('prev-frente');
    const prevR = document.getElementById('prev-reverso');
    const placeholder = document.getElementById('placeholder');
    const statusDiv = document.getElementById('status');
    const btnPDF = document.getElementById('btnPDF');

    const selectModal = document.getElementById('selectModal');
    const cropModal = document.getElementById('cropModal');
    const cropImg = document.getElementById('cropImg');

    /**
     * Muestra el modal de selección de origen (cámara o galería).
     * @param {string} s - Lado de la identificación a cargar ('frente' o 'reverso').
     */
    function openSelection(s) {
        side = s;
        selectModal.style.display = 'flex';
        // Añadir clase para la animación de entrada
        setTimeout(() => selectModal.classList.add('active'), 10);
    }

    /**
     * Oculta el modal de selección de origen.
     */
    function closeSelect() {
        selectModal.classList.remove('active');
        setTimeout(() => selectModal.style.display = 'none', 300);
    }

    /**
     * Simula el click en el input de archivo correspondiente.
     * @param {string} type - 'camera' o 'gallery'.
     */
    function choose(type) {
        closeSelect();
        document.getElementById(`${side}-${type}`).click();
    }

    // 🔄 Lógica de Carga de Imagen y Apertura del Crop Modal
    ['frente-camera', 'frente-gallery', 'reverso-camera', 'reverso-gallery']
    .forEach(id => {
        document.getElementById(id).onchange = e => {
            const file = e.target.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = ev => {
                cropImg.src = ev.target.result;
                cropModal.style.display = 'flex';
                setTimeout(() => cropModal.classList.add('active'), 10);
                
                // Inicializar Cropper.js después de que la imagen se ha cargado en el modal
                setTimeout(() => {
                    if (cropper) cropper.destroy();
                    rotation = 0; // Resetear rotación
                    cropper = new Cropper(cropImg, {
                        aspectRatio: 1.586, // Relación de aspecto estándar (e.g., ID, tarjeta de crédito)
                        viewMode: 1, // Restringe el área de recorte al tamaño del contenedor
                        autoCropArea: 0.9, // Área de auto-recorte inicial
                        movable: true,
                        zoomable: true,
                        rotatable: true,
                        scalable: false,
                    });
                }, 50);
            };
            reader.readAsDataURL(file);
        };
    });

    /**
     * Rota la imagen en el modal de recorte.
     */
    function rotateImg() {
        rotation += 90;
        cropper.rotateTo(rotation);
    }

    /**
     * Confirma el recorte, obtiene el DataURL y actualiza la UI.
     */
    function confirmCrop() {
        if (!cropper) return;
        
        // Obtener el canvas recortado con un tamaño de salida fijo para consistencia
        const croppedCanvas = cropper.getCroppedCanvas({
            width: 1000,
            imageSmoothingQuality: 'high'
        });
        
        // Convertir a DataURL (JPEG con calidad 85%)
        const dataURL = croppedCanvas.toDataURL('image/jpeg', 0.85);
        
        // Asignar al lado correspondiente
        if (side === 'frente') {
            frente = dataURL;
            prevF.src = dataURL;
        } else {
            reverso = dataURL;
            prevR.src = dataURL;
        }
        
        closeCrop();
        updateUI();
    }

    /**
     * Cierra y limpia el modal de recorte.
     */
    function closeCrop() {
        cropModal.classList.remove('active');
        setTimeout(() => {
            cropper?.destroy();
            cropper = null;
            rotation = 0;
            side = null;
            cropModal.style.display = 'none';
        }, 300);
    }

    /**
     * Actualiza la visibilidad de las imágenes de previsualización y el estado.
     */
    function updateUI() {
        const isComplete = frente && reverso;

        // Previsualización
        prevF.style.display = frente ? 'block' : 'none';
        prevR.style.display = reverso ? 'block' : 'none';
        placeholder.style.display = isComplete ? 'none' : 'block';

        // Botón y Estado
        btnPDF.disabled = !isComplete;
        statusDiv.setAttribute('data-complete', isComplete.toString());
        statusDiv.textContent = isComplete ? '✅ Listo para generar PDF' : '⚠️ Falta una imagen';
    }

    // ⬇️ Generación de PDF
    btnPDF.onclick = () => {
        if (!frente || !reverso) return;

        // Dimensiones del PDF A4
        // Ancho: 210 mm, Alto: 297 mm
        const pdf = new jsPDF({
            orientation: 'portrait',
            unit: 'mm',
            format: 'a4'
        });

        // Dimensiones estándar de una ID (CR-80): 85.60 mm x 53.98 mm.
        const idWidth = 85.6;
        const idHeight = 54;
        const marginX = (210 - (idWidth * 2) - 10) / 2; // Margen para centrar dos IDs, con 10mm de espacio entre ellas
        const startY = 20; // Posición vertical inicial

        // 1. Frente (Posicionado a la izquierda)
        pdf.addImage(frente, 'JPEG', marginX, startY, idWidth, idHeight);
        
        // 2. Reverso (Posicionado a la derecha)
        pdf.addImage(reverso, 'JPEG', marginX + idWidth + 10, startY, idWidth, idHeight);
        
        // Guardar el archivo
        pdf.save('Identificacion_Generada.pdf');
    };
    
    // Ejecutar al inicio para asegurar el estado inicial
    document.addEventListener('DOMContentLoaded', updateUI);
</script>

</body>
</html>
