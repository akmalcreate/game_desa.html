# game_desa.html
perduli lingkungan tanpa sampah
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kurir Sampah Desa: Tiny Planet</title>
    <style>
        body { margin: 0; padding: 0; background-color: #b3e5fc; display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100vh; font-family: sans-serif; overflow: hidden; }
        #ui { position: absolute; top: 20px; background: rgba(255, 255, 255, 0.85); padding: 10px 20px; border-radius: 30px; font-weight: bold; box-shadow: 0 4px 10px rgba(0,0,0,0.1); z-index: 10; }
        canvas { border-radius: 50%; box-shadow: 0 20px 40px rgba(0,0,0,0.3); background: #a5d6a7; }
        .kontrol { margin-top: 15px; color: #333; font-size: 14px; font-weight: bold; text-shadow: 1px 1px 0 #fff; }
    </style>
</head>
<body>

    <div id="ui">🗑️ Sampah: <span id="skor">0</span></div>
    
    <canvas id="planetCanvas" width="500" height="500"></canvas>
    
    <div class="kontrol">Gunakan **Tombol A / D** atau **Panah Kiri / Kanan**</div>

    <script>
        const canvas = document.getElementById("planetCanvas");
        const ctx = canvas.getContext("2d");
        const skorEl = document.getElementById("skor");

        let skor = 0;
        let sudutDunia = 0;
        const kecepatanRotasi = 0.05;
        const radiusPlanet = 200;
        const pusatX = 250;
        const pusatY = 430; // Pusat rotasi di bawah layar

        const keys = {};

        // Data Objek Desa (Rumah, Pohon, Sampah)
        const objekDesa = [
            { sudut: 0.2, tipe: 'rumah', warna: '#e57373' }, // Rumah pink (dari screenshot)
            { sudut: 1.0, tipe: 'sampah', emoji: '🍂', diambil: false }, // Daun
            { sudut: 1.5, tipe: 'pohon' },
            { sudut: 2.3, tipe: 'sampah', emoji: '🛍️', diambil: false }, // Kantong
            { sudut: 3.1, tipe: 'posronda' },
            { sudut: 4.4, tipe: 'sampah', emoji: '🧃', diambil: false }, // Kotak minum
            { sudut: 5.0, tipe: 'rumah', warna: '#64b5f6' }, // Rumah biru
            { sudut: 5.7, tipe: 'sampah', emoji: '🍾', diambil: false }  // Botol
        ];

        window.addEventListener("keydown", e => keys[e.key.toLowerCase()] = true);
        window.addEventListener("keyup", e => keys[e.key.toLowerCase()] = false);

        // --- FUNGSI UNTUK MENGGAMBAR KARAKTER 2D (VEKTOR MURNI) ---
        function drawPlayer(x, y) {
            ctx.save();
            ctx.translate(x, y);

            // Kaki (Celana Oranye)
            ctx.fillStyle = "#ff5722";
            ctx.fillRect(-6, 0, 4, 8); // Kaki kiri
            ctx.fillRect(2, 0, 4, 8);  // Kaki kanan

            // Badan (Baju Oranye)
            ctx.fillStyle = "#ff7043";
            ctx.fillRect(-8, -12, 16, 14);
            
            // Sabuk Hitam
            ctx.fillStyle = "#333";
            ctx.fillRect(-8, -2, 16, 2);

            // Kepala (Warna Kulit)
            ctx.fillStyle = "#ffcc80";
            ctx.beginPath();
            ctx.arc(0, -18, 6, 0, Math.PI * 2);
            ctx.fill();

            // Mata (Hitam)
            ctx.fillStyle = "#000";
            ctx.fillRect(-3, -20, 2, 2);
            ctx.fillRect(1, -20, 2, 2);

            // Topi Caping (Cokelat)
            ctx.fillStyle = "#a1887f";
            ctx.beginPath();
            ctx.moveTo(-12, -22); // Pinggir kiri
            ctx.lineTo(0, -32);   // Puncak
            ctx.lineTo(12, -22);  // Pinggir kanan
            ctx.closePath();
            ctx.fill();
            
            // Detail Topi (Garis)
            ctx.strokeStyle = "#8d6e63";
            ctx.lineWidth = 1;
            ctx.stroke();

            ctx.restore();
        }

        function update() {
            // Kontrol: A/D memutar dunia di bawah karakter
            if (keys["arrowleft"] || keys["a"]) sudutDunia -= kecepatanRotasi;
            if (keys["arrowright"] || keys["d"]) sudutDunia += kecepatanRotasi;

            if (sudutDunia < 0) sudutDunia += Math.PI * 2;
            if (sudutDunia > Math.PI * 2) sudutDunia -= Math.PI * 2;

            // Logika Pungut: Karakter dianggap diam di sudut paling atas (-Math.PI/2)
            const sudutKarakter = -Math.PI / 2;
            
            objekDesa.forEach(obj => {
                if (obj.tipe === 'sampah' && !obj.diambil) {
                    let sudutAbsolut = obj.sudut + sudutDunia;
                    sudutAbsolut = Math.atan2(Math.sin(sudutAbsolut), Math.cos(sudutAbsolut));
                    
                    if (Math.abs(sudutAbsolut - sudutKarakter) < 0.15) {
                        obj.diambil = true;
                        skor++;
                        skorEl.innerText = skor;
                        
                        // Respawn sampah
                        setTimeout(() => {
                            obj.diambil = false;
                            obj.sudut = Math.random() * Math.PI * 2;
                        }, 5000);
                    }
                }
            });
        }

        function draw() {
            ctx.fillStyle = "#a5d6a7"; // Warna langit/latar
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // 1. Gambar Inti Planet
            ctx.beginPath();
            ctx.arc(pusatX, pusatY, radiusPlanet, 0, Math.PI * 2);
            ctx.fillStyle = "#81c784"; // Warna tanah
            ctx.fill();
            ctx.lineWidth = 8;
            ctx.strokeStyle = "#5d4037"; // Warna pinggiran tanah
            ctx.stroke();

            // 2. Gambar Objek Desa yang Berputar
            objekDesa.forEach(obj => {
                if (obj.tipe === 'sampah' && obj.diambil) return;

                let sudutRender = obj.sudut + sudutDunia;
                let posX = pusatX + Math.cos(sudutRender) * radiusPlanet;
                let posY = pusatY + Math.sin(sudutRender) * radiusPlanet;

                // Hanya gambar objek yang terlihat di area atas planet
                if (posY < pusatY + 80) {
                    ctx.save();
                    ctx.translate(posX, posY);
                    ctx.rotate(sudutRender + Math.PI / 2);

                    if (obj.tipe === 'rumah') {
                        ctx.fillStyle = obj.warna;
                        ctx.fillRect(-20, -30, 40, 30);
                        ctx.fillStyle = "#b94a24"; // Atap
                        ctx.beginPath();
                        ctx.moveTo(-25, -30); ctx.lineTo(0, -45); ctx.lineTo(25, -30);
                        ctx.fill();
                    } else if (obj.tipe === 'pohon') {
                        ctx.fillStyle = "#5d4037"; ctx.fillRect(-4, -15, 8, 15); // Batang
                        ctx.fillStyle = "#2e7d32"; ctx.beginPath(); ctx.arc(0, -25, 15, 0, Math.PI * 2); ctx.fill(); // Daun
                    } else if (obj.tipe === 'sampah') {
                        ctx.font = "24px Arial"; ctx.fillText(obj.emoji, -12, -12);
                    } else if (obj.tipe === 'posronda') {
                        ctx.fillStyle = "#795548"; ctx.fillRect(-15, -25, 30, 25);
                        ctx.fillStyle = "#3e2723"; ctx.fillRect(-18, -25, 36, 5);
                    }

                    ctx.restore();
                }
            });

            // --- 3. GAMBAR KARAKTER (DI ATAS PLANET) ---
            // Karakter digambar terakhir agar menimpa planet dan objek.
            const charX = pusatX;
            const charY = pusatY - radiusPlanet; // Berdiri tepat di atas tanah

            drawPlayer(charX, charY);
        }

        function gameLoop() {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }

        // Jalankan game
        gameLoop();
    </script>
</body>
</html>
