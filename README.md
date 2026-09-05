<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Panel de Transmisión - TBB Progreso</title>
    <style>
        * { box-sizing: border-box; font-family: 'Segoe UI', system-ui, sans-serif; }
        body {
            background-color: #0f172a;
            color: #f8fafc;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 20px;
        }
        .panel-card {
            background-color: #1e293b;
            border: 1px solid #334155;
            border-radius: 16px;
            padding: 32px 24px;
            width: 100%;
            max-width: 500px;
            box-shadow: 0 20px 30px rgba(0,0,0,0.4);
            text-align: center;
        }
        h1 { font-size: 1.4rem; margin-bottom: 6px; }
        p.subtitulo { color: #94a3b8; font-size: 0.9rem; margin-top: 0; margin-bottom: 24px; }
        .estado-box {
            background-color: #0f172a;
            border-radius: 8px;
            padding: 12px;
            margin-bottom: 20px;
            font-size: 0.85rem;
            word-break: break-all;
            border: 1px solid #334155;
            text-align: left;
        }
        .estado-box span { font-weight: bold; color: #38bdf8; display: block; margin-bottom: 4px; }
        input[type="text"] {
            width: 100%;
            padding: 14px;
            background-color: #0f172a;
            border: 1px solid #475569;
            border-radius: 8px;
            color: #fff;
            font-size: 0.95rem;
            margin-bottom: 16px;
        }
        input[type="text"]:focus { outline: none; border-color: #38bdf8; }
        .btn {
            width: 100%;
            padding: 14px;
            border-radius: 8px;
            font-weight: 700;
            font-size: 1rem;
            cursor: pointer;
            border: none;
            transition: opacity 0.2s;
            margin-bottom: 10px;
        }
        .btn:disabled { opacity: 0.5; cursor: not-allowed; }
        .btn-activar { background-color: #22c55e; color: #fff; }
        .btn-apagar { background-color: #ef4444; color: #fff; }
        #mensaje { font-size: 0.9rem; margin-top: 15px; font-weight: 600; min-height: 24px; }
    </style>
</head>
<body>

    <div class="panel-card">
        <h1>Control de Transmisión</h1>
        <p class="subtitulo">TBB Progreso de Adoración</p>

        <div class="estado-box">
            <span>Transmisión activa actual:</span>
            <div id="url-actual">Cargando estado...</div>
        </div>

        <input type="text" id="inputUrl" placeholder="Pega aquí el enlace de Facebook">

        <button class="btn btn-activar" id="btnGuardar" onclick="actualizarTransmision('set')">
            🔴 Activar en Vivo en la Web
        </button>

        <button class="btn btn-apagar" id="btnBorrar" onclick="actualizarTransmision('clear')">
            ⏹️ Apagar / Finalizar Transmisión
        </button>

        <div id="mensaje"></div>
    </div>

    <script>
        // Asegúrate de colocar aquí tu enlace terminado en /exec
        const API_URL = "https://script.google.com/macros/s/AKfycbyclXRBDoIn9UhIA17EYgtKq31SMuINopxxlTIvPJK3AnBfQA9ndTjn9iyyosEufP5i/exec";

        // Función que limpia el link (quita ?rdid=..., m.facebook, etc.)
        function limpiarUrlFacebook(url) {
            if (!url) return "";
            url = url.trim();
            
            // Busca si viene el ID numérico del video
            const coincidencia = url.match(/\/videos\/(\d+)/) || url.match(/[?&]v=(\d+)/);
            if (coincidencia && coincidencia[1]) {
                return `https://www.facebook.com/taberelprogreso/videos/${coincidencia[1]}/`;
            }

            // Si no tiene el patrón anterior, elimina parámetros que inician con ? o #
            return url.split('?')[0].split('#')[0];
        }

        async function consultarEstado() {
            const visor = document.getElementById("url-actual");
            if (!API_URL || API_URL.includes("PEGA_AQUI")) {
                visor.innerText = "⚠️ Falta configurar la variable API_URL en el código.";
                visor.style.color = "#f59e0b";
                return;
            }

            try {
                const res = await fetch(`${API_URL}?t=${Date.now()}`);
                const data = await res.json();
                if (data.url && data.url.trim().length > 0) {
                    visor.innerText = data.url;
                    visor.style.color = "#4ade80";
                } else {
                    visor.innerText = "Ninguna (Sección en vivo apagada)";
                    visor.style.color = "#94a3b8";
                }
            } catch (err) {
                visor.innerText = "No se pudo conectar con Apps Script. Verifica la URL.";
                visor.style.color = "#ef4444";
            }
        }

        async function actualizarTransmision(accion) {
            const input = document.getElementById("inputUrl");
            const btnGuardar = document.getElementById("btnGuardar");
            const btnBorrar = document.getElementById("btnBorrar");
            const mensaje = document.getElementById("mensaje");

            if (!API_URL || API_URL.includes("PEGA_AQUI")) {
                mensaje.innerText = "Error: Debes poner tu enlace de Google Apps Script en API_URL.";
                mensaje.style.color = "#ef4444";
                return;
            }

            let urlFinal = "";
            if (accion === "set") {
                const textoPegado = input.value.trim();
                if (!textoPegado.startsWith("http")) {
                    mensaje.innerText = "Por favor pega un enlace válido (que empiece con https://).";
                    mensaje.style.color = "#ef4444";
                    return;
                }
                urlFinal = limpiarUrlFacebook(textoPegado);
            }

            btnGuardar.disabled = true;
            btnBorrar.disabled = true;
            mensaje.innerText = "Actualizando señal en la web...";
            mensaje.style.color = "#38bdf8";

            try {
                const endpoint = `${API_URL}?action=set&url=${encodeURIComponent(urlFinal)}&t=${Date.now()}`;
                
                // Petición al conector de Google Apps Script
                await fetch(endpoint, { mode: "no-cors" });

                mensaje.innerText = accion === "set" ? "¡Transmisión activada exitosamente!" : "¡Transmisión apagada en la web!";
                mensaje.style.color = "#22c55e";
                input.value = "";
                
                // Espera 2 segundos y actualiza el cuadro de estado
                setTimeout(consultarEstado, 2000);
            } catch (err) {
                mensaje.innerText = "Hubo un error de conexión al guardar.";
                mensaje.style.color = "#ef4444";
            } finally {
                btnGuardar.disabled = false;
                btnBorrar.disabled = false;
            }
        }

        consultarEstado();
    </script>
</body>
</html>
