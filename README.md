<!doctype html>

<html lang="fr">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Réviseur IA — Vérification de réponses</title>
<style>
/* Reset & fonts */
*{box-sizing:border-box;font-family:Inter, system-ui, sans-serif; margin:0; padding:0;}
body, html{height:100%;}/* Page d'accueil */ #home{display:flex;justify-content:center;align-items:center;height:100vh;flex-direction:column;background:linear-gradient(135deg, #7c3aed, #3b82f6, #06b6d4);background-size:400% 400%;animation: gradientBG 15s ease infinite; color:white;} #home h1{font-size:3rem;margin-bottom:20px;text-shadow: 2px 2px 10px rgba(0,0,0,0.3);} #home button{padding:12px 28px;border:none;border-radius:10px;background:white;color:#7c3aed;font-weight:bold;font-size:1.2rem;cursor:pointer;transition:0.3s;} #home button:hover{transform:scale(1.05);}

@keyframes gradientBG{ 0%{background-position:0% 50%;} 50%{background-position:100% 50%;} 100%{background-position:0% 50%;} }

/* Révisions */ #app{display:none;padding:20px;max-width:1200px;margin:0 auto;color:#f1f5f9;} textarea, input[type=text]{width:100%;padding:10px;border-radius:8px;border:1px solid rgba(255,255,255,0.2);background:#0b1220;color:white;margin-bottom:12px;} button.generate, button.check{background:#7c3aed;color:white;padding:10px 16px;border:none;border-radius:8px;cursor:pointer;transition:0.2s;} button.generate:hover, button.check:hover{opacity:0.9;} .question-card{background:#0b1220;padding:12px;margin:12px 0;border-radius:10px;border:1px solid rgba(255,255,255,0.1);} .answer-result{margin-top:6px;font-weight:bold;} .correct{color:#22c55e;} .wrong{color:#ef4444;} </style>

</head>
<body><div id="home">
<h1>Réviseur IA</h1>
<p>Colle ton cours, réponds aux questions et vérifie tes réponses automatiquement !</p>
<button id="startBtn">Commencer</button>
</div><div id="app">
<h2>Colle ton cours ici :</h2>
<textarea id="course" rows="8" placeholder="Colle le texte du cours ici..."></textarea>
<label>Nombre de questions :</label>
<input type="number" id="numQ" value="5" min="1" max="50">
<button class="generate" id="generateBtn">Générer les questions</button>
<div id="questionsContainer"></div>
</div><script>
// Page accueil
const startBtn = document.getElementById('startBtn');
const home = document.getElementById('home');
const app = document.getElementById('app');
startBtn.onclick = ()=>{ home.style.display='none'; app.style.display='block'; window.scrollTo(0,0); };

// Split sentences
function splitSentences(text){
  return text.replace(/\r/g,'').split(/[\n\.\?\!]+/).map(s=>s.trim()).filter(Boolean);
}

// Extract keywords simple
function extractKeywords(text){
  let words = text.toLowerCase().replace(/[^\w\s]/g,'').split(/\s+/).filter(w=>w.length>2);
  let freq = {};
  words.forEach(w=>freq[w]=(freq[w]||0)+1);
  let keys = Object.keys(freq).sort((a,b)=>freq[b]-freq[a]);
  return keys.slice(0,20);
}

// Generate questions (basic templates)
function generateQuestions(text, count){
  let sentences = splitSentences(text);
  let keywords = extractKeywords(text);
  let questions = [];
  for(let i=0;i<count;i++){
    let kw = keywords[i%keywords.length];
    questions.push({q:`Qu'est-ce que ${kw} ?`, ans:null});
  }
  return questions;
}

// Render questions
const generateBtn = document.getElementById('generateBtn');
const questionsContainer = document.getElementById('questionsContainer');
generateBtn.onclick = ()=>{
  questionsContainer.innerHTML='';
  let courseText = document.getElementById('course').value.trim();
  let count = parseInt(document.getElementById('numQ').value)||5;
  if(!courseText){ alert('Colle ton cours d\'abord'); return; }
  let questions = generateQuestions(courseText,count);
  questions.forEach((qObj,i)=>{
    let div = document.createElement('div'); div.className='question-card';
    div.innerHTML = `<div><strong>Q${i+1}:</strong> ${qObj.q}</div>
    <input type='text' placeholder='Ta réponse ici' class='answerInput'>
    <button class='check'>Vérifier</button>
    <div class='answer-result'></div>`;
    questionsContainer.appendChild(div);

    // check button
    div.querySelector('.check').onclick = ()=>{
      let answer = div.querySelector('.answerInput').value.trim().toLowerCase();
      let courseLower = courseText.toLowerCase();
      let resultDiv = div.querySelector('.answer-result');
      if(answer.length<1){ resultDiv.textContent='Tape une réponse'; resultDiv.className='answer-result'; return; }
      // simple matching: count common words
      let wordsAns = answer.split(/\s+/);
      let common = wordsAns.filter(w=>courseLower.includes(w));
      if(common.length / wordsAns.length > 0.5){
        resultDiv.textContent='✔ V Correct';
        resultDiv.className='answer-result correct';
      } else {
        resultDiv.textContent='✘ X Incorrect';
        resultDiv.className='answer-result wrong';
      }
    };
  });
};
</script></body>
</html>
