<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>📱 Generador de Credencial (Móvil)</title>

    <!-- jsPDF -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

    <!-- CropperJS -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/1.6.1/cropper.min.css" />
    <script src="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/1.6.1/cropper.min.js"></script>

    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 15px;
            background: #f2f3f4;
        }

        .container {
            background: white;
            padding: 20px;
            border-radius: 14px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.1);
        }

        h1 {
            font-size: 1.4rem;
            margin-bottom: 15px;
            text-align: center;
        }

        .upload-btn {
            display: block;
            width: 100%;
            background: #3498db;
            color: white;
            padding: 17px;
            margin-top: 18px;
            border-radius: 12px;
            text-align: center;
            font-size: 1.15rem;
            cursor: pointer;
            letter-spacing: 0.5px;
        }

        input[type="file"] {
            display: none;
        }

        .preview-area {
            margin-top: 25px;
            display: flex;
            flex-direction: column;
            gap: 20px;
        }

        .preview-area img {
            width: 100%;
            border-radius: 12px;
            border: 1px solid #ccc;
            padding: 5px;
            background: #fafafa;
        }

        #generate-pdf-btn {
            width: 100%;
            padding: 18px;
            margin-top: 30px;
            background: #27ae60;
            color: white;
            font-size: 1.2rem;
            border: none;
            border-radius: 12px;
        }

        #generate-pdf-btn:disabled {
            background: #95a5a6;
        }

        /* ===========================
           MODAL DE RECORTE
        ============================*/

        #crop-modal {
            display: none;
            position: fixed;
            top: 0; 
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.85);
            z-index: 9999;
            padding: 0;
            overflow: hidden;
            justify-content: center;
            align-items: center;
        }

        #crop-box {
            background: white;
            width: 100%;
            height: 100%;
            max-width: 450px;
            margin: auto;
            display: flex;
            flex-direction: column;
            border-radius: 0;
            padding: 12px;
        }

        .crop-container {
            flex: 1;
            overflow: auto;
            margin-bottom: 15px;
        }

        #image-to-crop {
            width: 100%;
        }

        .crop-buttons {
            position: sticky;
            bottom: 0;
            background: white;
            padding-bottom: 10px;
            padding-top: 10px;
        }

        .btn {
            width: 100%;
            padding: 15px;
            border-radius: 10px;
            margin-top: 10px;
            font-size: 1.1rem;
            border: none;
            color: white;
        }

        .btn-success { background: #2ecc71; }
        .btn-danger { background: #e74c3c; }
    </style>
</head>

<body>

<div class="container">

    <h1>📱 Generar PDF de Credencial</h1>

    <!-- Botón foto frente -->
    <label class="upload-btn">
        📸 Tomar / Subir Frente
        <input type="file" id="file-frente" accept="image/*" capture="environment">
    </label>

    <!-- Botón foto reverso -->
    <label class="upload-btn">
        📸 Tomar / Subir Reverso
        <input type="file" id="file-reverso" accept="image/*" capture="environment">
    </label>

    <!-- Vista previa -->
    <div class="preview-area">
        <img id="preview-frente" style="display:none;">
        <img id="preview-reverso" style="display:none;">
    </div>

    <button id="generate-pdf-btn" disabled>📥 Descargar PDF</button>
</div>

<!-- Modal para recortar -->
<div id="crop-modal">
    <div id="crop-box">
        <h3 style="text-align:center; margin-bottom:10px;">✂️ Ajustar Imagen</h3>

        <div class="crop-container">
            <img id="image-to-crop">
        </div>

        <div class="crop-buttons">
            <button id="crop-confirm-btn" class="btn btn-success">Confirmar</button>
            <button id="crop-cancel-btn" class="btn btn-danger">Cancelar</button>
        </div>
    </div>
</div>

<script>
let imgDataFrente = null;
let imgDataReverso = null;
let currentSide = null;
let cropper = null;

const { jsPDF } = window.jspdf;

document.getElementById("file-frente").onchange = () => loadAndCrop("frente");
document.getElementById("file-reverso").onchange = () => loadAndCrop("reverso");

const modal = document.getElementById("crop-modal");
const imgToCrop = document.getElementById("image-to-crop");

function loadAndCrop(side) {
    const fileInput = document.getElementById("file-" + side);
    const file = fileInput.files[0];
    if (!file) return;

    currentSide = side;

    const reader = new FileReader();
    reader.onload = e => {
        imgToCrop.src = e.target.result;
        modal.style.display = "flex";

        if (cropper) cropper.destroy();

        setTimeout(() => {
            cropper = new Cropper(imgToCrop, {
                aspectRatio: 1.586,
                viewMode: 1,
                autoCropArea: 1
            });
        }, 150);
    };

    reader.readAsDataURL(file);
}

document.getElementById("crop-confirm-btn").onclick = () => {
    cropper.getCroppedCanvas().toBlob(blob => {
        const reader = new FileReader();
        reader.onloadend = () => {
            const base64 = reader.result;

            const preview = document.getElementById("preview-" + currentSide);
            preview.src = base64;
            preview.style.display = "block";

            if (currentSide === "frente") imgDataFrente = base64;
            else imgDataReverso = base64;

            checkReady();
        };
        reader.readAsDataURL(blob);
    });

    modal.style.display = "none";
    cropper.destroy();
};

document.getElementById("crop-cancel-btn").onclick = () => {
    modal.style.display = "none";
    if (cropper) cropper.destroy();
};

function checkReady() {
    document.getElementById("generate-pdf-btn").disabled = !(imgDataFrente && imgDataReverso);
}

document.getElementById("generate-pdf-btn").onclick = () => {
    const pdf = new jsPDF("portrait", "mm", "a4");

    pdf.addImage(imgDataFrente, "JPEG", 10, 10, 90, 60);
    pdf.addImage(imgDataReverso, "JPEG", 110, 10, 90, 60);

    pdf.save("credencial.pdf");
};
</script>

</body>
</html>
