# game_desa.html
perduli lingkungan tanpa sampah
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kurir Sampah Desa: Tiny Planet</title>
    <style>
        body { margin: 0; padding: 0; background-color: #b3e5fc; display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100vh; font-family: sans-serif; overflow: hidden; }
        #ui { position: absolute; top: 20px; background: rgba(255, 255, 255, 0.85); padding: 10px 20px; border-radius: 30px; font-weight: bold; box-shadow: 0 4px 10px rgba(0,0,0,0.1); }
        canvas { border-radius: 50%; box-shadow: 0 20px 40px rgba(0,0,0,0.3); background: #81c784; }
        .kontrol { margin-top: 15px; color: #555; font-size: 14px; }
    </style>
</head>
<body>

    <div id="ui">🗑️ Sampah: <span id="skor">0</span></div>
    <canvas id="planetCanvas" width="500" height="500"></canvas>
    <div class="kontrol">Tekan **A / D** atau **Panah Kiri / Kanan** untuk memutar desa</div>

    <script>
        const canvas = document.getElementById("planetCanvas");
        const ctx = canvas.getContext("2d");
        const skorEl = document.getElementById("skor");

        let skor = 0;
        let sudutDunia = 0;
        const kecepatanRotasi = 0.04;
        const radiusPlanet = 200;
        const pusatX = 250;
        const pusatY = 430; // Pusat rotasi di bawah layar

        const keys = {};

        // --- UPDATE: DATA KARAKTER 2D (Sprite Pasukan Oranye) ---
        const spriteKarakter = new Image();
        // Menggunakan data URI Base64 untuk memastikan karakter langsung muncul tanpa file eksternal.
        spriteKarakter.src = "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAAsTAAALEwEAmpwYAAAAB3RJTUUH6AYbCxI7u73RhQAAAG1JREFUWMPtlLERgCAMRb8Le7gGu7gK67gGu7gGe7gGi7gCC3mBBO7gKEnAnwby8gshgYgI7uAsS9m7vWe9XNlZ9vK9vGdfeUov9Zis9PZg6Sg9g9X8g9Vcc9V6b7X+mOtc67S/9wNkiLgXidgHe9gS7GFLsAd7eAArXwshf9672QAAAABJRU5ErkJggg==";

        // Data Objek Desa (Sama seperti screenshotmu)
        const objekDesa = [
            { sudut: 0.2, tipe: 'rumah', warna: '#e57373' }, // Rumah pink
            { sudut: 1.0, tipe: 'sampah', emoji: '🍂', diambil: false }, // Daun
            { sudut: 1.5, tipe: 'pohon' },
            { sudut: 2.3, tipe: 'sampah', emoji: '🛍️', diambil: false }, // Kantong
            { sudut: 3.1, tipe: 'posronda' },
            { sudut: 4.4, tipe: 'sampah', emoji: '🧃', diambil: false }, // Kotak minum
            { sudut: 5.0, tipe: 'rumah', warna: '#64b5f6' } // Rumah biru
        ];

        window.addEventListener("keydown", e => keys[e.key.toLowerCase()] = true);
        window.addEventListener("keyup", e => keys[e.key.toLowerCase()] = false);

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
            ctx.fillStyle = "#a5d6a7";
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // 1. Gambar Inti Planet
            ctx.beginPath();
            ctx.arc(pusatX, pusatY, radiusPlanet, 0, Math.PI * 2);
            ctx.fillStyle = "#81c784";
            ctx.fill();
            ctx.lineWidth = 6;
            ctx.strokeStyle = "#5d4037";
            ctx.stroke();

            // 2. Gambar Objek Desa yang Berputar
            objekDesa.forEach(obj => {
                if (obj.tipe === 'sampah' && obj.diambil) return;

                let sudutRender = obj.sudut + sudutDunia;
                let posX = pusatX + Math.cos(sudutRender) * radiusPlanet;
                let posY = pusatY + Math.sin(sudutRender) * radiusPlanet;

                if (posY < pusatY + 50) {
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
                        ctx.font = "20px Arial"; ctx.fillText(obj.emoji, -10, -10);
                    } else if (obj.tipe === 'posronda') {
                        ctx.fillStyle = "#795548"; ctx.fillRect(-15, -25, 30, 25);
                    }

                    ctx.restore();
                }
            });

            // --- 3. UPDATE: GAMBAR KARAKTER ---
            // Karakter di-draw paling akhir agar menimpa planet dan objek.
            // Posisinya dikunci tepat di bagian tengah paling atas planet.
            const charX = pusatX;
            const charY = pusatY - radiusPlanet - 15; // Berdiri di permukaan tanah

            // Jika sprite sudah ter-load, gambar. Jika tidak, gambar lingkaran oranye sebagai backup.
            if (spriteKarakter.complete) {
                ctx.drawImage(spriteKarakter, charX - 16, charY - 16, 32, 32);
            } else {
                ctx.beginPath(); ctx.arc(charX, charY, 15, 0, Math.PI * 2); ctx.fillStyle = "#ff5722"; ctx.fill();
            }
        }

        function gameLoop() {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }

        // Jalankan game saat semua siap
        spriteKarakter.onload = () => { gameLoop(); };
        spriteKarakter.onerror = () => { gameLoop(); }; // Jalankan juga jika sprite gagal load
    </script>
</body>
</html>"data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAAsTAAALEwEAmpwYAAAAB3RJTUUH6AYbCxI7u73RhQAAAG1JREFUWMPtlLERgCAMRb8Le7gGu7gK67gGu7gGe7gGi7gCC3mBBO7gKEnAnwby8gshgYgI7uAsS9m7vWe9XNlZ9vK9vGdfeUov9Zis9PZg6Sg9g9X8g9Vcc9V6b7X+mOtc67S/9wNkiLgXidgHe9gS7GFLsAd7eAArXwshf9672QAAAABJRU5ErkJggg==";

        // 2. Data Objek Desa (Rumah Indonesia, Pohon, Sampah) yang tersebar di sekeliling planet (360 derajat)
        const objekDesa = [
            { sudut: 0, tipe: 'rumah', warna: '#e57373', nama: 'Rumah Pak RT' },
            { sudut: 0.5, tipe: 'sampah', emoji: '🍾', diambil: false },
            { sudut: 1.2, tipe: 'pohon' },
            { sudut: 1.8, tipe: 'rumah', warna: '#81c784', nama: 'Warung Bu Edi' },
            { sudut: 2.3, tipe: 'sampah', emoji: '🛍️', diambil: false },
            { sudut: 3.1, tipe: 'posronda' },
            { sudut: 3.8, tipe: 'pohon' },
            { sudut: 4.4, tipe: 'sampah', emoji: '🍂', diambil: false },
            { sudut: 5.0, tipe: 'rumah', warna: '#64b5f6', nama: 'Rumah Warga' },
            { sudut: 5.7, tipe: 'sampah', emoji: '🧃', diambil: false }
        ];

        window.addEventListener("keydown", e => keys[e.key.toLowerCase()] = true);
        window.addEventListener("keyup", e => keys[e.key.toLowerCase()] = false);

        function update() {
            // Karakter diam di tengah atas planet, dunianya yang berputar di bawah kaki karakter
            if (keys["arrowleft"] || keys["a"]) {
                sudutDunia -= kecepatanRotasi;
            }
            if (keys["arrowright"] || keys["d"]) {
                sudutDunia += kecepatanRotasi;
            }

            // Normalisasi sudut dunia agar berada di kisaran 0 hingga 2*PI
            if (sudutDunia < 0) sudutDunia += Math.PI * 2;
            if (sudutDunia > Math.PI * 2) sudutDunia -= Math.PI * 2;

            // Logika Pungut Sampah (Karakter berada di sudut posisi paling atas planet, yaitu -Math.PI / 2)
            const sudutKarakter = -Math.PI / 2;
            
            objekDesa.forEach(obj => {
                if (obj.tipe === 'sampah' && !obj.diambil) {
                    // Hitung posisi absolut objek saat dunia berputar
                    let sudutAbsolut = obj.sudut + sudutDunia;
                    // Normalisasi sudut absolut
                    sudutAbsolut = Math.atan2(Math.sin(sudutAbsolut), Math.cos(sudutAbsolut));
                    
                    // Jika objek berada sangat dekat dengan posisi karakter di atas planet
                    if (Math.abs(sudutAbsolut - sudutKarakter) < 0.15) {
                        obj.diambil = true;
                        skor++;
                        skorEl.innerText = skor;
                        
                        // Respawn sampah di tempat lain setelah beberapa saat
                        setTimeout(() => {
                            obj.diambil = false;
                            obj.sudut = Math.random() * Math.PI * 2;
                        }, 4000);
                    }
                }
            });
        }

        function draw() {
            // Bersihkan canvas dengan warna tanah/rumput planet
            ctx.fillStyle = "#a5d6a7";
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Gambar Lingkaran Inti Planet di bagian bawah canvas
            ctx.beginPath();
            ctx.arc(pusatX, pusatY, radiusPlanet, 0, Math.PI * 2);
            ctx.fillStyle = "#81c784";
            ctx.fill();
            ctx.lineWidth = 6;
            ctx.strokeStyle = "#5d4037";
            ctx.stroke();

            // Gambar Objek-Objek Desa yang Berputar mengelilingi planet
            objekDesa.forEach(obj => {
                if (obj.tipe === 'sampah' && obj.diambil) return;

                // Hitung posisi (X, Y) di permukaan lingkaran berdasarkan rotasi dunia
                let sudutRender = obj.sudut + sudutDunia;
                let posX = pusatX + Math.cos(sudutRender) * radiusPlanet;
                let posY = pusatY + Math.sin(sudutRender) * radiusPlanet;

                // Hanya gambar objek yang terlihat di area atas planet agar tidak membebani memori
                if (posY < pusatY + 50) {
                    ctx.save();
                    ctx.translate(posX, posY);
                    // Putar objek agar berdiri tegak tegak lurus dengan permukaan tanah bola
                    ctx.rotate(sudutRender + Math.PI / 2);

                    if (obj.tipe === 'rumah') {
                        // Rumah kecil khas 2D
                        ctx.fillStyle = obj.warna;
                        ctx.fillRect(-20, -30, 40, 30);
                        ctx.fillStyle = "#b94a24"; // Atap
                        ctx.beginPath();
                        ctx.moveTo(-25, -30);
                        ctx.lineTo(0, -45);
                        ctx.lineTo(25, -30);
                        ctx.fill();
                    } else if (obj.tipe === 'pohon') {
                        ctx.fillStyle = "#5d4037"; // Batang
                        ctx.fillRect(-4, -15, 8, 15);
                        ctx.fillStyle = "#2e7d32"; // Daun bulat
                        ctx.beginPath();
                        ctx.arc(0, -25, 15, 0, Math.PI * 2);
                        ctx.fill();
                    } else if (obj.tipe === 'posronda') {
                        ctx.fillStyle = "#795548";
                        ctx.fillRect(-15, -25, 30, 25);
                        ctx.fillStyle = "#3e2723";
                        ctx.fillRect(-18, -25, 36, 5);
                    } else if (obj.tipe === 'sampah') {
                        ctx.font = "20px Arial";
                        ctx.fillText(obj.emoji, -10, -10);
                    }

                    ctx.restore();
                }
            });

            // 3. Gambar Karakter Utama 2D tepat di bagian paling atas planet (Kamera Mengunci Karakter)
            const charX = pusatX;
            const charY = pusatY - radiusPlanet - 15; // Berdiri tepat di atas tanah lingkaran

            ctx.drawImage(spriteKarakter, charX - 16, charY - 16, 32, 32);
        }

        function gameLoop() {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }

        // Jalankan game setelah aset gambar siap
        spriteKarakter.onload = () => {
            gameLoop();
        };
    </script>
</body>
</html>
