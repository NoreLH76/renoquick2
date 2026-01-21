<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>RenoQuick - Estimatif Rénovation</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font-family: -apple-system, BlinkMacSystemFont, sans-serif; background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); min-height: 100vh; padding: 20px; }
        .container { max-width: 500px; margin: 0 auto; background: white; border-radius: 20px; box-shadow: 0 20px 40px rgba(0,0,0,0.1); overflow: hidden; }
        header { background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%); color: white; padding: 30px 20px; text-align: center; }
        h1 { font-size: 28px; font-weight: 800; margin-bottom: 10px; }
        .section { padding: 25px; border-bottom: 1px solid #eee; }
        .section:last-child { border-bottom: none; }
        label { display: block; font-weight: 600; margin-bottom: 10px; color: #333; }
        select, input { width: 100%; padding: 15px; border: 2px solid #e1e5e9; border-radius: 12px; font-size: 16px; margin-bottom: 20px; transition: border-color 0.3s; }
        select:focus, input:focus { outline: none; border-color: #4facfe; }
        .post-row { display: flex; align-items: center; margin-bottom: 15px; background: #f8f9ff; padding: 15px; border-radius: 12px; }
        .post-row input[type="checkbox"] { width: auto; margin-right: 15px; transform: scale(1.5); }
        .post-row input, .post-row select { flex: 1; margin-right: 10px; }
        .post-row input:last-child { margin-right: 0; }
        .measure-icon { font-size: 24px; color: #4facfe; margin-left: 10px; }
        button { background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%); color: white; border: none; padding: 18px 30px; border-radius: 15px; font-size: 18px; font-weight: 700; width: 100%; cursor: pointer; transition: transform 0.2s; }
        button:hover { transform: translateY(-2px); }
        .result { background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%); color: white; padding: 25px; text-align: center; display: none; }
        .result h3 { font-size: 24px; margin-bottom: 15px; }
        .total { font-size: 36px; font-weight: 800; margin: 10px 0; }
        .details { background: rgba(255,255,255,0.2); padding: 20px; border-radius: 12px; margin-top: 20px; font-size: 14px; }
        .details table { width: 100%; border-collapse: collapse; margin-top: 10px; }
        .details th, .details td { padding: 8px; text-align: left; }
        .details th { background: rgba(255,255,255,0.3); }
        @media (max-width: 480px) { .post-row { flex-direction: column; align-items: stretch; } .post-row input, .post-row select { margin-right: 0; margin-bottom: 10px; } }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>🏠 RenoQuick</h1>
            <p>Estimatif rénovation locative précis</p>
        </header>
        
        <div class="section">
            <label>Région</label>
            <select id="region">
                <option value="Normandie">Normandie (base)</option>
                <option value="Île-de-France">Île-de-France (+30%)</option>
                <option value="Sud-Ouest">Sud-Ouest (-10%)</option>
                <option value="PACA">PACA (+10%)</option>
                <option value="Autre">Autre région</option>
            </select>
        </div>
        
        <div class="section">
            <h3>Postes de travaux</h3>
            <div id="posts"></div>
        </div>
        
        <div class="section">
            <button onclick="calculate()">📊 Calculer estimatif</button>
        </div>
        
        <div class="result" id="result">
            <h3>✅ Votre estimatif</h3>
            <div class="total" id="total">0 € TTC</div>
            <div class="details">
                <div id="details"></div>
            </div>
            <button onclick="exportPDF()" style="margin-top: 20px; background: rgba(255,255,255,0.2);">📄 Exporter PDF</button>
        </div>
    </div>

    <script>
        const pricesData = {
            regions: {"Normandie":1.0,"Île-de-France":1.3,"Sud-Ouest":0.9,"PACA":1.1,"Autre":1.0},
            posts: {
                "Peinture murs/plafonds": {"base":30},
                "Revêtement sols PVC": {"base":45},
                "Carrelage zones humides": {"base":70},
                "Parquet stratifié": {"base":50},
                "Électricité (normes)": {"base":100},
                "Plomberie SDB/Cuisine": {"base":120},
                "Isolation murs": {"base":75},
                "Faux plafonds": {"base":70},
                "Changement fenêtres": {"base":400},
                "Cuisine équipée": {"base":350},
                "Salle de bain complète": {"base":650}
            },
            etats: {"Bon":0.8,"Moyen":1.0,"Mauvais":1.5,"Très mauvais":2.0}
        };

        const posts = Object.keys(pricesData.posts);
        const postsHtml = posts.map(post => `
            <div class="post-row">
                <input type="checkbox" id="post_${post.replace(/\s+/g,'_')}" onchange="togglePost('${post.replace(/\s+/g,'_')}')">
                <label for="post_${post.replace(/\s+/g,'_')}" style="font-weight: normal; flex: 1;">${post}</label>
                <input type="number" id="surface_${post.replace(/\s+/g,'_')}" placeholder="Surface m²" min="0" step="0.1" style="display:none;">
                <select id="etat_${post.replace(/\s+/g,'_')}" style="display:none;">
                    <option value="Bon">Bon</option>
                    <option value="Moyen" selected>Moyen</option>
                    <option value="Mauvais">Mauvais</option>
                    <option value="Très mauvais">Très mauvais</option>
                </select>
                <span class="measure-icon">📏</span>
            </div>
        `).join('');
        document.getElementById('posts').innerHTML = postsHtml;

        function togglePost(id) {
            const surface = document.getElementById(`surface_${id}`);
            const etat = document.getElementById(`etat_${id}`);
            if (surface.style.display === 'none') {
                surface.style.display = 'block';
                etat.style.display = 'block';
            } else {
                surface.style.display = 'none';
                etat.style.display = 'none';
                surface.value = '';
            }
        }

        function calculate() {
            const regionFactor = pricesData.regions[document.getElementById('region').value];
            let total = 0;
            let details = [];

            posts.forEach(post => {
                const id = post.replace(/\s+/g, '_');
                const checkbox = document.getElementById(`post_${id}`);
                if (checkbox.checked) {
                    const surface = parseFloat(document.getElementById(`surface_${id}`).value) || 0;
                    const etat = document.getElementById(`etat_${id}`).value;
                    const basePrice = pricesData.posts[post].base;
                    const price = basePrice * pricesData.etats[etat] * regionFactor * surface * 1.2; // TTC +20%
                    total += price;
                    details.push(`${post}: ${surface.toFixed(1)}m² × ${pricesData.etats[etat]} état = ${price.toFixed(0)}€`);
                }
            });

            document.getElementById('total').textContent = `${total.toFixed(0)} € TTC`;
            document.getElementById('details').innerHTML = details.map(d => `<div>${d}</div>`).join('');
            document.getElementById('result').style.display = 'block';
            document.getElementById('result').scrollIntoView({ behavior: 'smooth' });
        }

        function exportPDF() {
            window.print(); // Simple export imprimable
        }
    </script>
</body>
</html>
