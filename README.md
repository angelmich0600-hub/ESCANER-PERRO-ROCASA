<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>📷 Generador PDF de Identificación (Doble Cara) - Final Arreglado</title>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/1.6.1/cropper.min.css" />
    <script src="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/1.6.1/cropper.min.js"></script>
    
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f4f7f6;
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .container {
            background-color: #ffffff;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
            max-width: 900px;
            width: 100%;
        }

        h1 {
            color: #2c3e50;
            text-align: center;
            margin-bottom: 5px;
        }

        p.subtitle {
            text-align: center;
            color: #7f8c8d;
            margin-bottom: 30px;
            border-bottom: 2px solid #ecf0f1;
            padding-bottom: 15px;
        }

        .input-group {
            display: flex;
            justify-content: space-around;
            gap: 20px;
            margin-bottom: 30px;
        }

        /* Ahora es un div con onclick */
        .input-box {
            flex: 1;
            padding: 15px;
            border: 2px dashed #3498db;
            border-radius: 8px;
            text-align: center;
            background-color: #ecf0f1;
            cursor: pointer;
            transition: background-color 0.3s;
        }

        .input-box:hover {
            background-color: #e0e6e9;
            border-color: #2980b9;
        }

        input[type="file"] {
            display: none;
        }

        .preview-area {
            display: flex;
            justify-content: center;
            gap: 20px;
            min-height: 200px;
            border: 1px solid #ddd;
            padding: 15px;
            border-radius: 8px;
            margin-bottom: 20px;
        }

        .preview-area img {
            max-width: 45%;
            height: auto;
            border: 1px solid #ddd;
            object-fit: contain;
        }
        
        .placeholder {
            color: #95a5a6;
            display: flex;
            align-items: center;
            justify-content: center;
            font-style: italic;
        }

        #generate-pdf-btn {
            display: block;
            width: 100%;
            padding: 15px;
            background-color: #27ae60;
            color: white;
            border: none;
            border-radius: 8px;
            font-size: 1.2em;
            cursor: pointer;
            transition: background-color 0.3s;
        }

        #generate-pdf-btn:hover {
            background-color: #2ecc71;
        }
        
        #generate-pdf-btn:disabled {
            background-color: #95a5a6;
            cursor: not-allowed;
        }

        /* --- Estilos del Modal de Recorte --- */
        #crop-modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.8);
            z-index: 9999;
            justify-content: center;
            align-items: center;
        }
        
        .modal-content {
            background: white;
            padding: 20px;
            width: 90%;
            max-width: 600px;
            border-radius: 10px;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
        }

        #image-to-crop {
            display: block;
            max-width: 100%;
        }
        
        .modal-buttons button {
            padding: 10px 20px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-weight: bold;
            margin-top: 15px;
        }

        #crop-confirm-btn {
            background: #2ecc71;
            color: white;
        }

        #crop-cancel-btn {
            background: #e74c3c;
            color: white;
        }
        
        /* ESTILO BOTÓN DE ROTACIÓN */
        #rotate-btn {
            padding: 10px 20px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-weight: bold;
            background: #3498db;
            color: white;
            margin-top: 15px;
            display: block;
            width: 100%;
        }

        /* --- Estilos del NUEVO Modal de Selección --- */
        #selection-modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.6);
            z-index: 9998; 
            justify-content: center;
            align-items: center;
        }

        .modal-content-small {
            background: white;
            padding: 20px;
            width: 90%;
            max-width: 350px;
            border-radius: 10px;
            text-align: center;
        }

        .selection-options button {
            width: 100%;
            padding: 15px;
            margin: 10px 0;
            border: none;
            border-radius: 8px;
            font-size: 1.1em;
            cursor: pointer;
            font-weight: bold;
            transition: background-color 0.3s;
        }

        #select-camera-btn {
            background-color: #e74c3c;
            color: white;
        }

        #select-gallery-btn {
            background-color: #3498db;
            color: white;
        }

        #selection-cancel-btn {
            background: #bdc3c7;
            color: #2c3e50;
            margin-top: 15px;
            padding: 10px;
            border-radius: 5px;
            width: 100%;
            border: none;
            cursor: pointer;
        }
    </style>
</head>
<body>

    <div class="container">
        <h1>Generador de PDF para Identificación (Doble Cara)</h1>
        <p class="subtitle">Sube, recorta al tamaño de la INE y unifica ambas caras en una sola hoja PDF tamaño A4.</p>

        <div class="input-group">
            
            <div class="input-box" onclick="openSelectionMenu('frente')">
                📸 **Frente** (Toca para seleccionar origen)
                <input type="file" id="file-frente-camara" accept="image/*" capture="environment" onchange="handleFileInput(event, 'frente')" style="display:none;">
                <input type="file" id="file-frente-galeria" accept="image/*" onchange="handleFileInput(event, 'frente')" style="display:none;">
            </div>
            
            <div class="input-box" onclick="openSelectionMenu('reverso')">
                📸 **Reverso** (Toca para seleccionar origen)
                <input type="file" id="file-reverso-camara" accept="image/*" capture="environment" onchange="handleFileInput(event, 'reverso')" style="display:none;">
                <input type="file" id="file-reverso-galeria" accept="image/*" onchange="handleFileInput(event, 'reverso')" style="display:none;">
            </div>
        </div>
        <h2>Vista Previa (PDF Final)</h2>
        <div class="preview-area">
            <img id="preview-frente" src="" alt="Frente - Recortado" style="display:none;">
            <img id="preview-reverso" src="" alt="Reverso - Recortado" style="display:none;">
            <div id="placeholder" class="placeholder" style="display:flex;">Sube y recorta ambas imágenes.</div>
        </div>

        <button id="generate-pdf-btn" disabled>
            ⬇️ Generar y Compartir / Descargar PDF
        </button>
    </div>

    <div id="crop-modal">
        <div class="modal-content">
            <h2>✂️ Recortar Imagen</h2>
            <p>Ajusta el área para que coincida con tu credencial. El ratio es fijo (tarjeta estándar).</p>
            <div style="max-height: 400px; overflow: hidden; margin-bottom: 15px;">
                <img id="image-to-crop">
            </div>
            
            <button id="rotate-btn">🔄 Rotar 90°</button>
            <div class="modal-buttons">
                <button id="crop-confirm-btn">Confirmar Recorte</button>
                <button id="crop-cancel-btn">Cancelar</button>
            </div>
        </div>
    </div>
    
    <div id="selection-modal">
        <div class="modal-content-small">
            <h2>Elige una Opción</h2>
            <div class="selection-options">
                <button id="select-camera-btn">📷 Tomar Foto (Cámara)</button>
                <button id="select-gallery-btn">🖼️ Subir de Galería</button>
            </div>
            <button id="selection-cancel-btn">Cancelar</button>
        </div>
    </div>
    <script>
        // Variables globales
        let imgDataFrente = null;
        let imgDataReverso = null;
        let currentCropper = null;
        let currentSide = null;
        let rotated = 0; // Variable para rastrear la rotación acumulada
        let sideToSelect = null; // Variable para rastrear el lado en el menú de selección
        
        const { jsPDF } = window.jspdf;

        // Referencias del DOM (Actualizadas)
        const D = {
            cropModal: document.getElementById('crop-modal'),
            imageToCrop: document.getElementById('image-to-crop'),
            cropConfirmBtn: document.getElementById('crop-confirm-btn'),
            cropCancelBtn: document.getElementById('crop-cancel-btn'),
            generatePdfBtn: document.getElementById('generate-pdf-btn'),
            placeholder: document.getElementById('placeholder'),
            previewFrente: document.getElementById('preview-frente'),
            previewReverso: document.getElementById('preview-reverso'),
            rotateBtn: document.getElementById('rotate-btn'),
            
            // NUEVAS REFERENCIAS PARA EL MENÚ DE SELECCIÓN
            selectionModal: document.getElementById('selection-modal'),
            selectCameraBtn: document.getElementById('select-camera-btn'),
            selectGalleryBtn: document.getElementById('select-gallery-btn'),
            selectionCancelBtn: document.getElementById('selection-cancel-btn')
        };

        // --- MANEJADORES DE EVENTOS ---
        D.cropConfirmBtn.onclick = confirmCrop;
        D.cropCancelBtn.onclick = cancelCrop;
        D.generatePdfBtn.onclick = generatePDF;
        D.rotateBtn.onclick = () => { rotateImage(90) };
        
        // MANEJADORES DE EVENTOS PARA EL NUEVO MENÚ
        D.selectCameraBtn.onclick = () => triggerFileInput('camera');
        D.selectGalleryBtn.onclick = () => triggerFileInput('gallery');
        D.selectionCancelBtn.onclick = closeSelectionMenu;
        
        
        /**
         * Muestra el modal de selección y establece el lado actual.
         */
        function openSelectionMenu(side) {
            sideToSelect = side;
            D.selectionModal.style.display = 'flex';
        }

        /**
         * Oculta el modal de selección.
         */
        function closeSelectionMenu() {
            D.selectionModal.style.display = 'none';
            sideToSelect = null;
        }

        /**
         * Oculta el modal de selección y simula el click en el input de archivo apropiado.
         */
        function triggerFileInput(type) {
            if (!sideToSelect) return;

            closeSelectionMenu();

            let inputFileId;
            if (sideToSelect === 'frente') {
                inputFileId = (type === 'camera') ? 'file-frente-camara' : 'file-frente-galeria';
            } else {
                inputFileId = (type === 'camera') ? 'file-reverso-camara' : 'file-reverso-galeria';
            }
            
            // Simular el click en el input de archivo oculto
            document.getElementById(inputFileId).click();
        }

        /**
         * Maneja la subida del archivo y comienza el proceso de recorte.
         */
        function handleFileInput(event, side) {
            const file = event.target.files[0];
            // IMPORTANTE: Limpiar el valor del input después de tomar el archivo
            event.target.value = ''; 

            if (file) {
                if (!['image/jpeg', 'image/png'].includes(file.type)) {
                    alert('Formato de archivo no soportado. Por favor, sube una imagen JPEG o PNG.');
                    return;
                }

                const reader = new FileReader();
                reader.onload = (e) => {
                    currentSide = side;
                    rotated = 0; // Reinicia la rotación para cada nueva imagen
                    
                    if (currentCropper) currentCropper.destroy();
                    D.imageToCrop.src = e.target.result;
                    
                    D.cropModal.style.display = 'flex';
                    
                    // Inicializa Cropper.js
                    setTimeout(() => {
                         currentCropper = new Cropper(D.imageToCrop, {
                            aspectRatio: 1.586, // Ratio estándar de tarjeta de crédito/INE
                            viewMode: 1, 
                            dragMode: 'move',
                            autoCropArea: 0.9,
                        });
                    }, 50);
                };
                reader.readAsDataURL(file);
            }
        }
        
        /**
         * Rotar la imagen 90 grados usando Cropper.js
         */
        function rotateImage(degree) {
            if (currentCropper) {
                rotated += degree;
                currentCropper.rotateTo(rotated);
            }
        }


        /**
         * Confirma el recorte y actualiza las variables de datos y la vista previa.
         */
        function confirmCrop() {
            if (!currentCropper) return;

            // Obtiene la imagen recortada en formato base64 con un ancho de 500px para buena calidad
            const base64Data = currentCropper.getCroppedCanvas({
                imageSmoothingQuality: 'high',
                width: 500, 
            }).toDataURL('image/jpeg', 0.9);

            // Actualiza la variable de datos y la vista previa
            const previewElement = (currentSide === 'frente') ? D.previewFrente : D.previewReverso;
            previewElement.src = base64Data;
            previewElement.style.display = 'block';

            if (currentSide === 'frente') {
                imgDataFrente = base64Data;
            } else {
                imgDataReverso = base64Data;
            }

            // Cierra el modal y limpia el cropper
            closeCropModal();
            checkAndEnableButton();
        }

        /**
         * Cancela la operación de recorte y cierra el modal.
         */
        function cancelCrop() {
            closeCropModal();
        }

        function closeCropModal() {
            if (currentCropper) {
                currentCropper.destroy();
                currentCropper = null;
            }
            D.cropModal.style.display = 'none';
        }

        /**
         * Verifica el estado de las imágenes y habilita/deshabilita el botón.
         */
        function checkAndEnableButton() {
            if (imgDataFrente && imgDataReverso) {
                D.generatePdfBtn.disabled = false;
                D.placeholder.style.display = 'none';
            } else {
                D.generatePdfBtn.disabled = true;
                D.placeholder.style.display = 'flex';
            }
        }
        
        // --- FUNCIÓN PRINCIPAL DE GENERACIÓN Y COMPARTICIÓN (CORREGIDA) ---

        /**
         * Genera el PDF con las imágenes colocadas correctamente.
         * @returns {Promise<Blob>} Promesa que resuelve con el Blob del PDF.
         */
        function createPDFBlob() {
            return new Promise((resolve, reject) => {
                if (!imgDataFrente || !imgDataReverso) {
                    return reject(new Error("Faltan datos de imagen (frente o reverso)."));
                }
                
                const doc = new jsPDF('portrait', 'mm', 'a4');
                const margin = 15;
                const spacing = 5;
                const cardWidth = 85.6; // Ancho estándar INE en mm
                const cardHeight = 54; // Alto estándar INE en mm
                
                // Función para cargar una imagen Base64
                const loadImage = (data) => {
                    return new Promise((res, rej) => {
                        const img = new Image();
                        img.onload = () => res(img);
                        img.onerror = (e) => rej(e);
                        img.src = data;
                    });
                };

                // Cargar ambas imágenes antes de intentar añadirlas al PDF
                Promise.all([loadImage(imgDataFrente), loadImage(imgDataReverso)])
                    .then(([imgFrente, imgReverso]) => {
                        
                        // Colocar el FRENTE (arriba a la izquierda)
                        doc.addImage(imgFrente, 'JPEG', margin, margin, cardWidth, cardHeight);

                        // Colocar el REVERSO (arriba a la derecha, junto al frente)
                        const xReverso = margin + cardWidth + spacing;
                        const yReverso = margin;
                        
                        doc.addImage(imgReverso, 'JPEG', xReverso, yReverso, cardWidth, cardHeight);
                        
                        // Generar y resolver con el Blob
                        const pdfBlob = doc.output('blob');
                        resolve(pdfBlob);
                        
                    })
                    .catch(error => {
                        console.error("Error al cargar imágenes para PDF:", error);
                        reject(new Error("No se pudieron cargar las imágenes recortadas."));
                    });
            });
        }


        /**
         * Llama a la Web Share API para abrir el menú nativo del móvil.
         */
        async function sharePDF(pdfFile) {
            try {
                await navigator.share({
                    files: [pdfFile],
                    title: 'Identificación',
                    text: 'Documento generado con la app web.',
                });
                
                console.log('PDF compartido con éxito.');

            } catch (error) {
                if (error.name === 'AbortError') {
                    console.log('Compartir cancelado por el usuario.');
                } else {
                    console.error('Error al intentar compartir:', error);
                    alert('Fallo al compartir. El PDF será descargado automáticamente.');
                    // Fallback: si falla el compartir, descarga el archivo
                    // No propagamos el error para no detener generatePDF completamente, pero forzamos la descarga.
                    const fallbackBlob = new Blob([pdfFile], { type: 'application/pdf' });
                    savePDF(fallbackBlob, 'Identificacion_Doble_Cara.pdf');
                }
                // Se lanza un error solo si fue un error distinto a AbortError para no finalizar el proceso.
                if (error.name !== 'AbortError') {
                    throw error;
                }
            }
        }
        
        /**
         * Fuerza la descarga del Blob/File en el navegador.
         */
        function savePDF(blob, fileName) {
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = fileName;
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
        }

        /**
         * Función principal para generar, intentar compartir y descargar el PDF.
         */
        async function generatePDF() {
            if (D.generatePdfBtn.disabled) return;
            
            const fileName = 'Identificacion_Doble_Cara.pdf';
            
            try {
                D.generatePdfBtn.disabled = true; // Deshabilitar durante el proceso
                D.generatePdfBtn.textContent = '⏳ Generando PDF...';

                const pdfBlob = await createPDFBlob();

                // 1. Intentar compartir si el navegador lo soporta
                const fileToShare = new File([pdfBlob], fileName, { type: 'application/pdf' });
                
                if (navigator.canShare && navigator.canShare({ files: [fileToShare] })) {
                    
                    await sharePDF(fileToShare); 
                    
                } else {
                    // 2. Si no se puede compartir, forzar la descarga
                    savePDF(pdfBlob, fileName);
                    alert("Tu dispositivo/navegador no soporta la opción de compartir archivos. El PDF ha sido descargado automáticamente.");
                }
                
            } catch (error) {
                // Manejo de errores de createPDFBlob o errores serios de sharePDF
                alert("Ocurrió un error grave al generar o compartir el PDF. Revisa las imágenes.");
                console.error("Error en generatePDF:", error);
            } finally {
                D.generatePdfBtn.disabled = false;
                D.generatePdfBtn.textContent = '⬇️ Generar y Compartir / Descargar PDF';
            }
        }
        
        // Inicializa el estado del botón
        checkAndEnableButton();
    </script>

</body>
</html>
