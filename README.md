# game_desa.html
perduli lingkungan tanpa sampah
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Petualangan Bersih Desa</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: #a3d9c9;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            overflow: hidden;
        }
        #gameUi {
            color: #2c3e50;
            font-size: 18px;
            font-weight: bold;
            margin-bottom: 10px;
            text-align: center;
            background: rgba(255, 255, 255, 0.8);
            padding: 10px 20px;
            border-radius: 20px;
            box-shadow: 0 4px 6px id=rgba(0,0,0,0.1);
        }
        canvas {
            border: 6px solid #4a3728;
            border-radius: 15px;
            background-color: #81c784; /* Warna rumput desa */
            box-shadow: 0 10px 20px rgba(0,0,0,0.2);
            max-width: 100%;
        }
        .instruksi {
            margin-top: 15px;
            color: #555;
            font-size: 14px;
            text-align: center;
            max-width: 400px;
            line-height: 1.4;
        }
    </style>
</head>
<body>

    <div id="gameUi">
        🗑️ Sampah Terkumpul: <span id="skor">0</span> / 10 | 🏡 Desa Sukamaju
    </div>

    <canvas id="gameCanvas" width="800" height="600"></canvas>

    <div class="instruksi">
        <strong>Cara Bermain:</strong><br>
        Gunakan tombol <b>Panah (Arrow Keys)</b> atau <b>W, A, S, D</b> di keyboard untuk keliling desa dan memungut sampah yang berserakan!
    </div>

    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");
        const skorElemen = document.getElementById("skor");

        // State Game
        let skor = 0;
        const totalSampah = 10;

        // Karakter Pemain (Pak/Bu RT)
        const pemain = {
            x: canvas.width / 2,
            y: canvas.height / 2,
            radius: 15,
            speed: 4,
            warna: "#ff7043" // Baju oranye khas petugas kebersihan/gotong royong
        };

        // Ornamen Desa (Rumah, Pohon, Gapura)
        const elemenDesa = [
            { x: 100, y: 100, tipe: 'rumah', warna: '#e57373' },
            { x: 350, y: 80, tipe: 'rumah', warna: '#64b5f6' },
            { x: 650, y: 120, tipe: 'rumah', warna: '#fff176' },
            { x: 200, y: 450, tipe: 'pohon' },
            { x: 600, y: 400, tipe: 'pohon' },
            { x: 50, y: 300, tipe: 'pohon' },
            { x: 400, y: 520, tipe: 'posronda' }
        ];

        // Daftar Sampah
        let daftarSampah = [];
        const jenisSampah = [
            { teks: "🍾", nama: "Botol" },
            { teks: "🛍️", nama: "Kresek" },
            { teks: "🍂", nama: "Daun" }
        ];

        // Tombol Kontrol
        const tombol = {};

        // Inisialisasi Sampah secara acak di peta
        function gantiPosisiSampah(sampah) {
            sampah.x = Math.random() * (canvas.width - 60) + 30;
            sampah.y = Math.random() * (canvas.height - 60) + 30;
            // Memastikan sampah tidak muncul tepat di atas rumah
            elemenDesa.forEach(el => {
                let dist = Math.hypot(sampah.x - el.x, sampah.y - el.y);
                if (dist < 50) {
                    sampah.x += 60;
                }
            });
        }

        for (let i = 0; i < totalSampah; i++) {
            let info = jenisSampah[Math.floor(Math.random() * jenisSampah.length)];
            let sampah = { x: 0, y: 0, info: info };
            gantiPosisiSampah(sampah);
            daftarSampah.push(sampah);
        }

        // Event Listener Keyboard
        window.addEventListener("keydown", e => tombol[e.key.toLowerCase()] = true);
        window.addEventListener("keyup", e => tombol[e.key.toLowerCase()] = false);

        // Update Logika Game
        function update() {
            // Pergerakan pemain
            if (tombol["arrowup"] || tombol["w"]) pemain.y -= pemain.speed;
            if (tombol["arrowdown"] || tombol["s"]) pemain.y += pemain.speed;
            if (tombol["arrowleft"] || tombol["a"]) pemain.x -= pemain.speed;
            if (tombol["arrowright"] || tombol["d"]) pemain.x += pemain.speed;

            // Batas dinding canvas agar tidak keluar desa
            if (pemain.x < pemain.radius) pemain.x = pemain.radius;
            if (pemain.x > canvas.width - pemain.radius) pemain.x = canvas.width - pemain.radius;
            if (pemain.y < pemain.radius) pemain.y = pemain.radius;
            if (pemain.y > canvas.height - pemain.radius) pemain.y = canvas.height - pemain.radius;

            // Cek tabrakan dengan sampah (Pungut)
            daftarSampah.forEach(sampah => {
                let jarak = Math.hypot(pemain.x - sampah.x, pemain.y - sampah.y);
                if (jarak < pemain.radius + 15) {
                    // Pindahkan sampah ke lokasi baru (seolah-olah muncul sampah baru)
                    gantiPosisiSampah(sampah);
                    skor++;
                    skorElemen.innerText = skor;
                }
            });
        }

        // Menggambar Grafis ke Layar
        function draw() {
            // Bersihkan layar dengan warna dasar rumput desa
            ctx.fillStyle = "#81c784";
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Gambar Jalan Setapak Desa
            ctx.fillStyle = "#cfd8dc";
            ctx.fillRect(0, canvas.height / 2 - 25, canvas.width, 50);
            ctx.fillRect(canvas.width / 2 - 25, 0, 50, canvas.height);

            // Gambar Ornamen Desa (Rumah & Pohon)
            elemenDesa.forEach(el => {
                if (el.tipe === 'rumah') {
                    // Tembok Rumah
                    ctx.fillStyle = el.warna;
                    ctx.fillRect(el.x - 30, el.y - 20, 60, 50);
                    // Atap Rumah
                    ctx.fillStyle = "#d84315";
                    ctx.beginPath();
                    ctx.moveTo(el.x - 40, el.y - 20);
                    ctx.lineTo(el.x, el.y - 50);
                    ctx.lineTo(el.x + 40, el.y - 20);
                    ctx.fill();
                } else if (el.tipe === 'pohon') {
                    // Batang
                    ctx.fillStyle = "#5d4037";
                    ctx.fillRect(el.x - 5, el.y, 10, 30);
                    // Daun
                    ctx.fillStyle = "#2e7d32";
                    ctx.beginPath();
                    ctx.arc(el.x, el.y - 10, 20, 0, Math.PI * 2);
                    ctx.fill();
                } else if (el.tipe === 'posronda') {
                    // Pos Ronda Tradisional
                    ctx.fillStyle = "#8d6e63";
                    ctx.fillRect(el.x - 25, el.y - 20, 50, 40);
                    ctx.fillStyle = "#4e342e";
                    ctx.fillRect(el.x - 20, el.y - 40, 40, 20); // Atap ijuk
                }
            });

            // Gambar Sampah
            ctx.font = "20px Arial";
            ctx.textAlign = "center";
            ctx.textBaseline = "middle";
            daftarSampah.forEach(sampah => {
                ctx.fillText(sampah.info.teks, sampah.x, sampah.y);
            });

            // Gambar Pemain (Karakter Utama)
            ctx.beginPath();
            ctx.arc(pemain.x, pemain.x, pemain.radius, 0, Math.PI * 2); // Kepala/Badan atas
            ctx.fillStyle = pemain.warna;
            ctx.fill();
            ctx.closePath();
            
            // Topi Caping Petani / Karakter Desa
            ctx.beginPath();
            ctx.fillStyle = "#ffb74d";
            ctx.arc(pemain.x, pemain.y, pemain.radius + 2, Math.PI, 0);
            ctx.fill();
            ctx.closePath();
        }

        // Game Loop
        function gameLoop() {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }

        // Jalankan Game
        gameLoop();
    </script>
</body>
</html>
