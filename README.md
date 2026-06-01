<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MateVida App</title>
    <style>
        * { box-sizing: border-box; }
        body { font-family: 'Segoe UI', Arial, sans-serif; background: #eef2ff; margin: 0; padding: 10px; }
        .titulo-app { font-size: clamp(2rem, 8vw, 3.5rem); color: #4f46e5; text-align: center; margin: 20px 0; }
        #menu { display: flex; gap: 10px; padding: 10px; background: #fff; border-radius: 12px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); margin-bottom: 20px; justify-content: center; }
        .card { background: white; padding: 25px; border-radius: 20px; box-shadow: 0 10px 30px rgba(0,0,0,0.1); text-align: center; max-width: 600px; margin: auto; }
        .avatar-box { width: 150px; height: 150px; margin: 0 auto 15px auto; border-radius: 50%; border: 4px solid #4f46e5; overflow: hidden; background: #e0e7ff; display: flex; align-items: center; justify-content: center; }
        .avatar-box svg { width: 75%; height: 75%; }
        .msg-box { font-weight: bold; padding: 10px; border-radius: 8px; margin: 10px 0; font-size: 1.2em; height: 50px; }
        #timer { font-size: 2.5em; color: #d97706; font-weight: bold; margin: 10px 0; }
        button { padding: 15px 30px; border: none; cursor: pointer; border-radius: 10px; background: #4f46e5; color: white; font-size: 18px; font-weight: bold; margin: 5px; }
        input { padding: 12px; border-radius: 8px; border: 2px solid #ddd; width: 80%; font-size: 24px; text-align: center; margin: 10px 0; }
        .input-perfil { font-size: 16px; padding: 8px; width: 60%; }
        /* IMPORTANTE: Ajuste para móviles */
@media (max-width: 768px) {
    .sidebar { width: 100%; height: auto; position: relative; } /* El menú se vuelve una barra superior */
    .main { margin-left: 0; width: 100%; } /* El contenido ocupa todo el ancho */
    .grid { grid-template-columns: repeat(2, 1fr); } /* Módulos de 2 en 2 en móviles */
}
    
    </style>
</head>
<body>

<h1 class="titulo-app">MateVida</h1>
<div id="menu">
    <button onclick="abrirModulo('tablas')">📊 Tablas</button>
    <button onclick="abrirModulo('perfil')">👤 Perfil</button>
</div>
<div id="contenido"></div>

<script>
    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();

    function reproducirTono(tipo) {
        if (audioCtx.state === 'suspended') {
            audioCtx.resume();
        }

        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.connect(gain);
        gain.connect(audioCtx.destination);

        const tiempoActual = audioCtx.currentTime;

        if (tipo === 'ok') {
            osc.type = 'triangle';
            osc.frequency.setValueAtTime(523.25, tiempoActual); 
            osc.frequency.setValueAtTime(659.25, tiempoActual + 0.1); 
            gain.gain.setValueAtTime(0.3, tiempoActual);
            gain.gain.exponentialRampToValueAtTime(0.01, tiempoActual + 0.3);
            osc.start(tiempoActual);
            osc.stop(tiempoActual + 0.3);
        } else if (tipo === 'fail') {
            osc.type = 'sawtooth';
            osc.frequency.setValueAtTime(180, tiempoActual);
            osc.frequency.linearRampToValueAtTime(100, tiempoActual + 0.2);
            gain.gain.setValueAtTime(0.2, tiempoActual);
            gain.gain.exponentialRampToValueAtTime(0.01, tiempoActual + 0.25);
            osc.start(tiempoActual);
            osc.stop(tiempoActual + 0.25);
        } else if (tipo === 'win') {
            osc.type = 'sine';
            osc.frequency.setValueAtTime(392.00, tiempoActual); 
            osc.frequency.setValueAtTime(523.25, tiempoActual + 0.15); 
            osc.frequency.setValueAtTime(659.25, tiempoActual + 0.3); 
            osc.frequency.setValueAtTime(783.99, tiempoActual + 0.45); 
            gain.gain.setValueAtTime(0.3, tiempoActual);
            gain.gain.exponentialRampToValueAtTime(0.01, tiempoActual + 0.8);
            osc.start(tiempoActual);
            osc.stop(tiempoActual + 0.8);
        }
    }

    let preguntas = [], index = 0, aciertos = 0, tiempo = 100, intervalo;

    if(!localStorage.getItem('nombreUsuario')) localStorage.setItem('nombreUsuario', 'Estudiante');
    if(!localStorage.getItem('recordPuntaje')) localStorage.setItem('recordPuntaje', '0');

    const avatarSVG = `
        <div class="avatar-box">
            <svg viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                <circle cx="50" cy="50" r="40" fill="#4f46e5"/>
                <rect x="35" y="35" width="30" height="25" rx="5" fill="#ffffff"/>
                <circle cx="43" cy="45" r="4" fill="#4f46e5"/>
                <circle cx="57" cy="45" r="4" fill="#4f46e5"/>
                <rect x="45" y="52" width="10" height="3" rx="1.5" fill="#4f46e5"/>
                <rect x="48" y="25" width="4" height="10" fill="#ffffff"/>
                <circle cx="50" cy="23" r="3" fill="#fbbf24"/>
            </svg>
        </div>`;

    function abrirModulo(modulo){
        clearInterval(intervalo);
        const c = document.getElementById("contenido");

        if (modulo === 'perfil') {
            let nombreActual = localStorage.getItem('nombreUsuario');
            let recordActual = localStorage.getItem('recordPuntaje');

            c.innerHTML = `
                <div class="card">
                    ${avatarSVG}
                    <h2>👤 Perfil del Estudiante</h2>
                    <p style="font-size: 1.2em; color: #4b5563;">¡Bienvenido a tu panel de progreso!</p>
                    
                    <div style="text-align: left; background: #f3f4f6; padding: 15px; border-radius: 10px; margin: 20px 0;">
                        <p><strong>Nombre:</strong> <span id="lblNombre">${nombreActual}</span></p>
                        <p><strong>Puntaje Máximo:</strong> ${recordActual} / 100</p>
                        <p><strong>Rango:</strong> ${parseInt(recordActual) >= 80 ? '👑 Maestro de las Tablas' : '✏️ Aprendiz Activo'}</p>
                    </div>

                    <div style="margin-bottom: 20px;">
                        <input id="inputNombre" class="input-perfil" type="text" placeholder="Cambiar tu nombre..." value="${nombreActual}">
                        <button onclick="guardarNombre()" style="padding: 8px 15px; font-size: 14px;">💾 Guardar</button>
                    </div>

                    <button onclick="abrirModulo('tablas')">📊 Ir a Tablas</button>
                </div>`;
        } else {
            c.innerHTML = `
                <div class="card">
                    ${avatarSVG}
                    <div id="msg" class="msg-box">¡Hola! ¿Listo para el desafío?</div>
                    <button onclick="iniciarLeccion()">🚀 Iniciar Desafío (100s)</button>
                    <div id="area"></div>
                </div>`;
        }
    }

    function guardarNombre() {
        let nuevoNombre = document.getElementById("inputNombre").value.trim();
        if(nuevoNombre !== "") {
            localStorage.setItem('nombreUsuario', nuevoNombre);
            document.getElementById("lblNombre").innerText = nuevoNombre;
            alert("¡Nombre guardado con éxito!");
        }
    }

    function iniciarLeccion(){
        if (audioCtx.state === 'suspended') { audioCtx.resume(); }
        
        preguntas = []; index = 0; aciertos = 0; tiempo = 100;
        for(let i=0; i<15; i++){
            let a = Math.floor(Math.random()*11)+2;
            let b = Math.floor(Math.random()*10)+1;
            preguntas.push({ q: a + " × " + b + " = ?", a: a*b });
        }
        
        document.getElementById("area").innerHTML = `
            <div id="timer">100s</div>
            <h2 id="q">${preguntas[0].q}</h2>
            <input id="r" type="number" inputmode="numeric" autofocus><br>
            <button onclick="responder()">Enviar</button>`;
        
        document.getElementById("r").addEventListener("keyup", function(event) {
            if (event.key === "Enter") {
                responder();
            }
        });

        document.getElementById("r").focus();

        intervalo = setInterval(() => {
            tiempo--;
            document.getElementById("timer").innerText = tiempo + "s";
            if(tiempo <= 0) fin();
        }, 1000);
    }

    function responder(){
        let val = document.getElementById("r").value;
        let msg = document.getElementById("msg");
        if(val === "") return;
        
        if(parseInt(val) === preguntas[index].a){
            aciertos++;
            reproducirTono('ok'); 
            msg.innerText = "¡Excelente, sigue así! 🌟";
            msg.style.color = "green";
        } else {
            reproducirTono('fail'); 
            let vozErr = new SpeechSynthesisUtterance("Vuelve a intentarlo");
            vozErr.lang = "es-ES";
            window.speechSynthesis.speak(vozErr);

            msg.innerText = "¡Oh no! Vuelve a intentarlo 🔄";
            msg.style.color = "red";
        }
        
        index++;
        if(index < preguntas.length) {
            document.getElementById("q").innerText = preguntas[index].q;
            document.getElementById("r").value = "";
            document.getElementById("r").focus();
        } else { fin(); }
    }

    function fin(){
        clearInterval(intervalo);
        reproducirTono('win'); 
        
        let puntajeFinal = Math.round((aciertos/15)*100);
        let recordActual = parseInt(localStorage.getItem('recordPuntaje'));
        
        if(puntajeFinal > recordActual) {
            localStorage.setItem('recordPuntaje', puntajeFinal.toString());
        }

        document.getElementById("area").innerHTML = `
            <h2>🏁 Juego Finalizado</h2>
            <p style="font-size: 1.4em;">Puntaje obtenido: <strong>${puntajeFinal} / 100</strong></p>
            <button onclick="abrirModulo('tablas')">Volver al inicio</button>`;
    }

    abrirModulo('tablas');
</script>
</body>
</html>
