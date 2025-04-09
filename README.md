# Lucas-pharma.github.io
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Quiz Pharma Ultimate</title>
    <!-- Bibliothèque pour générer le PDF -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <style>
        :root {
            --pharma-green: #2e8b57;
            --pharma-blue: #4682b4;
            --pharma-light: #f0f9f0;
        }
        body {
            font-family: 'Segoe UI', sans-serif;
            background: var(--pharma-light);
            color: #333;
            max-width: 700px;
            margin: 0 auto;
            padding: 20px;
            line-height: 1.6;
        }
        h1 {
            color: var(--pharma-green);
            text-align: center;
            font-size: 2.2rem;
        }
        .question {
            background: white;
            padding: 25px;
            border-radius: 15px;
            margin: 25px 0;
            box-shadow: 0 5px 15px rgba(0,0,0,0.1);
            border-left: 5px solid var(--pharma-green);
        }
        .options div {
            padding: 15px;
            margin: 12px 0;
            background: #e8f5e9;
            border-radius: 10px;
            cursor: pointer;
            transition: all 0.3s;
            font-weight: 500;
        }
        .options div:hover {
            background: var(--pharma-green);
            color: white;
            transform: translateY(-3px);
        }
        .selected {
            background: var(--pharma-green) !important;
            color: white !important;
            font-weight: bold;
            position: relative;
        }
        .selected::after {
            content: "✓";
            position: absolute;
            right: 15px;
        }
        button {
            background: var(--pharma-green);
            color: white;
            border: none;
            padding: 14px 28px;
            border-radius: 50px;
            font-size: 1.1rem;
            font-weight: 600;
            cursor: pointer;
            display: block;
            margin: 30px auto;
            transition: all 0.3s;
        }
        button:hover {
            background: #3cb371;
            transform: scale(1.05);
        }
        button:disabled {
            background: #cccccc;
            cursor: not-allowed;
            transform: none;
        }
        #result {
            display: none;
            animation: fadeIn 0.5s ease-out;
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .job-card {
            background: white;
            padding: 25px;
            border-radius: 15px;
            margin: 30px 0;
            box-shadow: 0 5px 20px rgba(0,0,0,0.1);
        }
        .job-title {
            color: var(--pharma-green);
            font-size: 1.8rem;
            margin-bottom: 15px;
            display: flex;
            align-items: center;
            gap: 15px;
        }
        .job-img {
            width: 80px;
            height: 80px;
            object-fit: contain;
        }
        .job-section {
            margin-bottom: 20px;
        }
        .job-section h4 {
            color: var(--pharma-blue);
            font-size: 1.2rem;
            margin-bottom: 10px;
            border-bottom: 2px dashed #eee;
            padding-bottom: 5px;
        }
        .progress-container {
            width: 100%;
            background: #e0e0e0;
            border-radius: 10px;
            margin: 20px 0;
            height: 8px;
        }
        .progress-bar {
            height: 100%;
            background: linear-gradient(90deg, var(--pharma-green), var(--pharma-blue));
            border-radius: 10px;
            width: 0%;
            transition: width 0.4s ease;
        }
        .question-count {
            text-align: right;
            color: var(--pharma-green);
            font-weight: 600;
        }
        .action-buttons {
            display: flex;
            gap: 15px;
            justify-content: center;
            flex-wrap: wrap;
        }
        .confetti {
            position: fixed;
            width: 10px;
            height: 10px;
            background-color: #f00;
            opacity: 0;
            z-index: 9999;
            animation: confetti 5s ease-out;
        }
        @keyframes confetti {
            0% { transform: translateY(0) rotate(0deg); opacity: 1; }
            100% { transform: translateY(500px) rotate(360deg); opacity: 0; }
        }
    </style>
</head>
<body>
    <h1>💊 Quiz Ultime Pharma 🚀</h1>
    
    <div id="quiz">
        <div class="question-count">Question <span id="current-question">1</span>/10</div>
        <div class="progress-container">
            <div class="progress-bar" id="progress-bar"></div>
        </div>
        <div id="question-container"></div>
        <button id="next-btn" disabled>Suivant →</button>
    </div>
    
    <div id="result">
        <div class="job-card">
            <h2 class="job-title" id="result-title"></h2>
            <img id="result-img" class="job-img" src="" alt="">
            <p id="result-desc"></p>
        </div>
        
        <div class="job-card" id="pdf-content">
            <h3 class="job-title">📋 Fiche Métier</h3>
            <div class="job-section">
                <h4>🎯 Missions</h4>
                <ul id="job-missions"></ul>
            </div>
            <div class="job-section">
                <h4>🛠️ Compétences</h4>
                <ul id="job-skills"></ul>
            </div>
            <div class="job-section">
                <h4>💰 Salaire</h4>
                <p id="job-salary"></p>
            </div>
        </div>
        
        <div class="action-buttons">
            <button id="restart-btn">🔄 Recommencer</button>
            <button id="pdf-btn">⬇️ Télécharger PDF</button>
            <button id="share-btn">📤 Partager</button>
        </div>
    </div>

    <script>
        // Questions complètes (10)
        const questions = [
            {
                question: "🚀 Quel héros de science-fiction es-tu ?",
                options: [
                    "Spock (Star Trek) - Logique et précis",
                    "Le Docteur (Doctor Who) - Curieux et empathique",
                    "Katniss (Hunger Games) - Combatif et pragmatique",
                    "Tony Stark (Iron Man) - Innovateur et entrepreneur"
                ],
                scores: ["scientifique", "hospitalier", "humanitaire", "officine"]
            },
            {
                question: "💼 Ton environnement idéal ?",
                options: [
                    "Labo de recherche high-tech",
                    "Hôpital dynamique",
                    "Mission humanitaire à l'étranger",
                    "Officine de quartier conviviale"
                ],
                scores: ["scientifique", "hospitalier", "humanitaire", "officine"]
            },
            {
                question: "🧪 Ton super-pouvoir pharma ?",
                options: [
                    "Inventer des médicaments révolutionnaires",
                    "Diagnostiquer n'importe quelle maladie",
                    "Soigner avec des moyens limités",
                    "Conseiller parfaitement chaque patient"
                ],
                scores: ["scientifique", "hospitalier", "humanitaire", "officine"]
            },
            {
                question: "📚 Ton approche de travail ?",
                options: [
                    "Méthodique et précis",
                    "Collaboratif et humain",
                    "Adaptable et réactif",
                    "Commercial et relationnel"
                ],
                scores: ["scientifique", "hospitalier", "humanitaire", "officine"]
            },
            {
                question: "🌍 En mission à l'étranger, tu serais...",
                options: [
                    "Dans un labo pharmaceutique innovant",
                    "Dans un hôpital de renom",
                    "En zone de crise humanitaire",
                    "Dans une pharmacie locale"
                ],
                scores: ["scientifique", "hospitalier", "humanitaire", "officine"]
            },
            {
                question: "⏱️ Face à une urgence...",
                options: [
                    "Je consulte les protocoles scientifiques",
                    "Je coordonne l'équipe médicale",
                    "J'improvise avec les moyens du bord",
                    "Je rassure le patient avant tout"
                ],
                scores: ["scientifique", "hospitalier", "humanitaire", "officine"]
            },
            {
                question: "💡 Ta source de motivation ?",
                options: [
                    "Découvrir quelque chose de nouveau",
                    "Faire partie d'une équipe soudée",
                    "Aider les plus démunis",
                    "Créer du lien avec les patients"
                ],
                scores: ["scientifique", "hospitalier", "humanitaire", "officine"]
            },
            {
                question: "🏆 Ton rêve ultime ?",
                options: [
                    "Gagner un prix Nobel",
                    "Diriger un grand hôpital",
                    "Changer le système de santé mondial",
                    "Devenir le pharmacien le plus apprécié"
                ],
                scores: ["scientifique", "hospitalier", "humanitaire", "officine"]
            },
            {
                question: "📱 Ton appli préférée ?",
                options: [
                    "Un logiciel d'analyse moléculaire",
                    "Un outil de gestion hospitalière",
                    "Une plateforme de crowdfunding médical",
                    "Un réseau social pour conseils santé"
                ],
                scores: ["scientifique", "hospitalier", "humanitaire", "officine"]
            },
            {
                question: "🎵 Ta musique de travail ?",
                options: [
                    "Son labo high-tech",
                    "Bruits d'ambulances et d'urgence",
                    "Chants traditionnels du monde",
                    "Bruissement de l'officine"
                ],
                scores: ["scientifique", "hospitalier", "humanitaire", "officine"]
            }
        ];

        // Fiches métiers complètes
        const jobs = {
            "scientifique": {
                title: "🔬 Chercheur Pharmaceutique",
                img: "https://cdn-icons-png.flaticon.com/512/1995/1995485.png",
                desc: "Tu es l'innovateur qui repousse les limites de la science médicale !",
                missions: [
                    "Développer de nouveaux médicaments",
                    "Conduire des essais cliniques",
                    "Publier des recherches scientifiques"
                ],
                skills: [
                    "Maîtrise des techniques de labo",
                    "Esprit analytique aiguisé",
                    "Connaissances en biologie moléculaire"
                ],
                salary: "40 000€ - 80 000€"
            },
            "hospitalier": {
                title: "🏥 Pharmacien Hospitalier",
                img: "https://cdn-icons-png.flaticon.com/512/2785/2785482.png",
                desc: "Tu es le maillon essentiel des équipes médicales !",
                missions: [
                    "Gérer les stocks de médicaments",
                    "Préparer des chimiothérapies",
                    "Former le personnel soignant"
                ],
                skills: [
                    "Gestion des urgences",
                    "Connaissances thérapeutiques",
                    "Travail en équipe pluridisciplinaire"
                ],
                salary: "35 000€ - 60 000€"
            },
            "humanitaire": {
                title: "🌍 Pharmacien Humanitaire",
                img: "https://cdn-icons-png.flaticon.com/512/2785/2785529.png",
                desc: "Tu apportes les soins là où personne ne va !",
                missions: [
                    "Distribuer des médicaments en zones critiques",
                    "Former des professionnels locaux",
                    "Organiser des campagnes de prévention"
                ],
                skills: [
                    "Adaptabilité extrême",
                    "Gestion de crise",
                    "Connaissances en santé publique"
                ],
                salary: "Variable (1 500€ - 3 000€/mois)"
            },
            "officine": {
                title: "💊 Pharmacien d'Officine",
                img: "https://cdn-icons-png.flaticon.com/512/2063/2063150.png",
                desc: "Tu es le conseiller santé de référence !",
                missions: [
                    "Conseiller les patients",
                    "Délivrer les ordonnances",
                    "Gérer l'approvisionnement"
                ],
                skills: [
                    "Excellente connaissance des médicaments",
                    "Relation client exceptionnelle",
                    "Gestion commerciale"
                ],
                salary: "35 000€ - 70 000€"
            }
        };

        // Variables du quiz
        let currentQuestion = 0;
        let scores = {
            "scientifique": 0,
            "hospitalier": 0,
            "humanitaire": 0,
            "officine": 0
        };
        let selectedOption = null;

        // Afficher la question actuelle
        function showQuestion() {
            const q = questions[currentQuestion];
            let html = `<div class="question"><h3>${q.question}</h3><div class="options">`;
            
            q.options.forEach((opt, i) => {
                html += `<div onclick="selectOption(${i})">${opt}</div>`;
            });
            
            document.getElementById('question-container').innerHTML = html + `</div></div>`;
            
            // Mettre à jour la progression
            document.getElementById('current-question').textContent = currentQuestion + 1;
            document.getElementById('progress-bar').style.width = `${((currentQuestion + 1) / questions.length) * 100}%`;
            
            // Réinitialiser la sélection
            selectedOption = null;
            document.getElementById('next-btn').disabled = true;
        }

        // Sélectionner une option
        window.selectOption = function(index) {
            // Désélectionner l'option précédente
            if (selectedOption !== null) {
                document.querySelectorAll('.options div')[selectedOption].classList.remove('selected');
            }
            
            // Sélectionner la nouvelle option
            selectedOption = index;
            document.querySelectorAll('.options div')[index].classList.add('selected');
            document.getElementById('next-btn').disabled = false;
        };

        // Créer des confettis
        function createConfetti() {
            const colors = ['#2e8b57', '#4682b4', '#ffd700', '#ff6347', '#9370db'];
            
            for (let i = 0; i < 100; i++) {
                const confetti = document.createElement('div');
                confetti.classList.add('confetti');
                confetti.style.backgroundColor = colors[Math.floor(Math.random() * colors.length)];
                confetti.style.left = Math.random() * 100 + 'vw';
                confetti.style.width = Math.random() * 10 + 5 + 'px';
                confetti.style.height = Math.random() * 10 + 5 + 'px';
                confetti.style.animationDuration = Math.random() * 3 + 2 + 's';
                document.body.appendChild(confetti);
                
                // Supprimer après l'animation
                setTimeout(() => {
                    confetti.remove();
                }, 5000);
            }
        }

        // Générer PDF
        function generatePDF() {
            const { jsPDF } = window.jspdf;
            const element = document.getElementById('pdf-content');
            
            html2canvas(element).then(canvas => {
                const imgData = canvas.toDataURL('image/png');
                const pdf = new jsPDF('p', 'mm', 'a4');
                const imgProps = pdf.getImageProperties(imgData);
                const pdfWidth = pdf.internal.pageSize.getWidth();
                const pdfHeight = (imgProps.height * pdfWidth) / imgProps.width;
                
                pdf.addImage(imgData, 'PNG', 0, 0, pdfWidth, pdfHeight);
                pdf.save('fiche-metier-pharmacien.pdf');
            });
        }

        // Partager le résultat
        function shareResult() {
            const job = document.getElementById('result-title').textContent;
            const desc = document.getElementById('result-desc').textContent;
            
            if (navigator.share) {
                navigator.share({
                    title: 'Mon résultat du quiz pharma : ' + job,
                    text: desc,
                    url: window.location.href
                }).catch(err => {
                    alert("Partage annulé : " + err);
                });
            } else {
                alert("Fonction de partage non disponible sur ce navigateur. Copiez le lien manuellement.");
            }
        }

        // Afficher le résultat
        function showResult() {
            // Trouver le score max
            let maxScore = 0;
            let result = 'scientifique';
            
            for (const [type, score] of Object.entries(scores)) {
                if (score > maxScore) {
                    maxScore = score;
                    result = type;
                }
            }
            
            // Afficher le résultat
            const job = jobs[result];
            document.getElementById('result-title').textContent = job.title;
            document.getElementById('result-img').src = job.img;
            document.getElementById('result-desc').textContent = job.desc;
            
            // Afficher la fiche métier
            document.getElementById('job-missions').innerHTML = 
                job.missions.map(m => `<li>${m}</li>`).join('');
            document.getElementById('job-skills').innerHTML = 
                job.skills.map(s => `<li>${s}</li>`).join('');
            document.getElementById('job-salary').textContent = job.salary;
            
            // Basculer sur l'écran de résultat
            document.getElementById('quiz').style.display = 'none';
            document.getElementById('result').style.display = 'block';
            
            // Lancer les confettis
            createConfetti();
        }

        // Écouteurs d'événements
        document.getElementById('next-btn').addEventListener('click', () => {
            // Enregistrer le score
            const q = questions[currentQuestion];
            scores[q.scores[selectedOption]]++;
            
            // Question suivante ou résultat
            currentQuestion++;
            if (currentQuestion < questions.length) {
                showQuestion();
            } else {
                showResult();
            }
        });

        document.getElementById('restart-btn').addEventListener('click', () => {
            currentQuestion = 0;
            scores = {
                "scientifique": 0,
                "hospitalier": 0,
                "humanitaire": 0,
                "officine": 0
            };
            document.getElementById('quiz').style.display = 'block';
            document.getElementById('result').style.display = 'none';
            showQuestion();
        });

        document.getElementById('pdf-btn').addEventListener('click', generatePDF);
        document.getElementById('share-btn').addEventListener('click', shareResult);

        // Démarrer le quiz
        showQuestion();
    </script>
</body>
</html>
