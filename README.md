# pecobaan
import React, { useState, useEffect } from 'react';
import { Sword, Shield, Heart, Star, Zap, Trophy, BookOpen } from 'lucide-react';

const AlgebraRPG = () => {
  const [gameState, setGameState] = useState('menu'); // menu, game, victory, defeat
  const [player, setPlayer] = useState({
    name: 'Hero',
    level: 1,
    exp: 0,
    expToNext: 100,
    hp: 100,
    maxHp: 100,
    attack: 10,
    defense: 5,
    gold: 0,
    skills: {
      basic: true,
      intermediate: false,
      advanced: false
    }
  });

  const [enemy, setEnemy] = useState(null);
  const [currentQuestion, setCurrentQuestion] = useState(null);
  const [userAnswer, setUserAnswer] = useState('');
  const [feedback, setFeedback] = useState('');
  const [battleLog, setBattleLog] = useState([]);
  const [stage, setStage] = useState(1);

  const stages = {
    1: { name: 'Forest of Basic Algebra', difficulty: 'basic', enemies: ['Goblin Penjumlah', 'Slime Pengurangan'] },
    2: { name: 'Cave of Equations', difficulty: 'intermediate', enemies: ['Orc Persamaan', 'Troll Variabel'] },
    3: { name: 'Mountain of Quadratics', difficulty: 'advanced', enemies: ['Dragon Kuadrat', 'Phoenix Fungsi'] }
  };

  const generateQuestion = (difficulty) => {
    let question, answer, explanation;
    
    if (difficulty === 'basic') {
      const types = ['simple', 'combine', 'solve'];
      const type = types[Math.floor(Math.random() * types.length)];
      
      if (type === 'simple') {
        const a = Math.floor(Math.random() * 10) + 1;
        const b = Math.floor(Math.random() * 10) + 1;
        question = `Berapakah nilai dari ${a}x jika x = ${b}?`;
        answer = a * b;
        explanation = `${a}x dengan x = ${b} adalah ${a} × ${b} = ${answer}`;
      } else if (type === 'combine') {
        const a = Math.floor(Math.random() * 5) + 1;
        const b = Math.floor(Math.random() * 5) + 1;
        question = `Sederhanakan: ${a}x + ${b}x`;
        answer = a + b;
        explanation = `${a}x + ${b}x = ${a + b}x, koefisiennya adalah ${answer}`;
      } else {
        const a = Math.floor(Math.random() * 5) + 2;
        const b = Math.floor(Math.random() * 10) + 5;
        answer = Math.floor(b / a);
        const result = a * answer;
        question = `Selesaikan: ${a}x = ${result}`;
        explanation = `x = ${result} ÷ ${a} = ${answer}`;
      }
    } else if (difficulty === 'intermediate') {
      const types = ['linear', 'twovar', 'distribute'];
      const type = types[Math.floor(Math.random() * types.length)];
      
      if (type === 'linear') {
        const a = Math.floor(Math.random() * 5) + 2;
        const b = Math.floor(Math.random() * 10) + 5;
        const c = Math.floor(Math.random() * 10) + 10;
        answer = Math.floor((c - b) / a);
        question = `Selesaikan: ${a}x + ${b} = ${c}`;
        explanation = `${a}x = ${c} - ${b} = ${c - b}, maka x = ${answer}`;
      } else if (type === 'twovar') {
        const a = Math.floor(Math.random() * 3) + 1;
        const b = Math.floor(Math.random() * 3) + 1;
        question = `Sederhanakan: ${a}x + ${b}y - x + 2y. Berapa koefisien x?`;
        answer = a - 1;
        explanation = `(${a}x - x) + (${b}y + 2y) = ${a - 1}x + ${b + 2}y, koefisien x adalah ${answer}`;
      } else {
        const a = Math.floor(Math.random() * 3) + 2;
        const b = Math.floor(Math.random() * 3) + 1;
        const c = Math.floor(Math.random() * 3) + 1;
        question = `Jabarkan: ${a}(x + ${b}). Berapa koefisien x?`;
        answer = a;
        explanation = `${a}(x + ${b}) = ${a}x + ${a * b}, koefisien x adalah ${answer}`;
      }
    } else { // advanced
      const types = ['quadratic', 'factor', 'complete'];
      const type = types[Math.floor(Math.random() * types.length)];
      
      if (type === 'quadratic') {
        const roots = [2, 3, 4, 5];
        const r1 = roots[Math.floor(Math.random() * roots.length)];
        const r2 = roots[Math.floor(Math.random() * roots.length)];
        const b = -(r1 + r2);
        const c = r1 * r2;
        question = `Selesaikan x² ${b >= 0 ? '+' : ''}${b}x ${c >= 0 ? '+' : ''}${c} = 0. Berapa salah satu nilai x? (bulat)`;
        answer = Math.min(r1, r2);
        explanation = `Faktorisasi: (x - ${r1})(x - ${r2}) = 0, maka x = ${r1} atau x = ${r2}`;
      } else if (type === 'factor') {
        const a = Math.floor(Math.random() * 3) + 2;
        const b = Math.floor(Math.random() * 3) + 2;
        question = `Faktorkan: x² + ${a + b}x + ${a * b}. Berapa konstanta di faktor pertama?`;
        answer = Math.min(a, b);
        explanation = `x² + ${a + b}x + ${a * b} = (x + ${a})(x + ${b})`;
      } else {
        const h = Math.floor(Math.random() * 4) + 1;
        const k = Math.floor(Math.random() * 5) + 1;
        question = `Titik puncak parabola y = (x - ${h})² + ${k} ada di x = ?`;
        answer = h;
        explanation = `Bentuk vertex y = (x - h)² + k memiliki puncak di (h, k) = (${h}, ${k})`;
      }
    }
    
    return { question, answer, explanation };
  };

  const generateEnemy = (stageNum) => {
    const stageData = stages[stageNum];
    const enemyName = stageData.enemies[Math.floor(Math.random() * stageData.enemies.length)];
    const baseHp = 50 + stageNum * 30;
    const baseAtk = 5 + stageNum * 5;
    
    return {
      name: enemyName,
      hp: baseHp,
      maxHp: baseHp,
      attack: baseAtk,
      difficulty: stageData.difficulty
    };
  };

  const startBattle = () => {
    const newEnemy = generateEnemy(stage);
    setEnemy(newEnemy);
    setCurrentQuestion(generateQuestion(newEnemy.difficulty));
    setBattleLog([`${newEnemy.name} muncul!`]);
    setFeedback('');
    setUserAnswer('');
    setGameState('game');
  };

  const checkAnswer = () => {
    if (!userAnswer) return;
    
    const numAnswer = parseFloat(userAnswer);
    const isCorrect = Math.abs(numAnswer - currentQuestion.answer) < 0.01;
    
    if (isCorrect) {
      const damage = player.attack + Math.floor(Math.random() * 10);
      const newEnemyHp = enemy.hp - damage;
      
      setEnemy({ ...enemy, hp: Math.max(0, newEnemyHp) });
      setBattleLog([...battleLog, `✓ Benar! Kamu menyerang ${damage} damage!`]);
      setFeedback(`Correct! ${currentQuestion.explanation}`);
      
      if (newEnemyHp <= 0) {
        const expGain = 50 + stage * 25;
        const goldGain = 20 + stage * 10;
        const newExp = player.exp + expGain;
        let newLevel = player.level;
        let newExpToNext = player.expToNext;
        let leveledUp = false;
        
        if (newExp >= player.expToNext) {
          newLevel++;
          newExpToNext = 100 * newLevel;
          leveledUp = true;
        }
        
        setPlayer({
          ...player,
          exp: newExp >= player.expToNext ? newExp - player.expToNext : newExp,
          expToNext: newExpToNext,
          level: newLevel,
          maxHp: leveledUp ? player.maxHp + 20 : player.maxHp,
          hp: leveledUp ? player.maxHp + 20 : player.hp,
          attack: leveledUp ? player.attack + 5 : player.attack,
          defense: leveledUp ? player.defense + 2 : player.defense,
          gold: player.gold + goldGain,
          skills: {
            ...player.skills,
            intermediate: newLevel >= 5 ? true : player.skills.intermediate,
            advanced: newLevel >= 10 ? true : player.skills.advanced
          }
        });
        
        setBattleLog([...battleLog, `✓ Benar! Kamu menyerang ${damage} damage!`, `${enemy.name} dikalahkan! +${expGain} EXP, +${goldGain} Gold${leveledUp ? ' LEVEL UP!' : ''}`]);
        
        setTimeout(() => {
          if (stage < 3 && newLevel >= stage * 4) {
            setStage(stage + 1);
            setBattleLog([`Area baru terbuka: ${stages[stage + 1].name}!`]);
          }
          setGameState('menu');
        }, 3000);
      } else {
        setTimeout(() => enemyAttack(), 1500);
      }
    } else {
      setFeedback(`Salah! ${currentQuestion.explanation}`);
      setBattleLog([...battleLog, `✗ Salah! ${enemy.name} menyerang balik!`]);
      setTimeout(() => enemyAttack(), 1500);
    }
    
    setUserAnswer('');
  };

  const enemyAttack = () => {
    const damage = Math.max(1, enemy.attack - player.defense + Math.floor(Math.random() * 5));
    const newHp = player.hp - damage;
    
    setPlayer({ ...player, hp: Math.max(0, newHp) });
    setBattleLog(prev => [...prev, `${enemy.name} menyerang ${damage} damage!`]);
    
    if (newHp <= 0) {
      setBattleLog(prev => [...prev, 'Kamu kalah! EXP dan Gold dikurangi.']);
      setPlayer(p => ({
        ...p,
        hp: p.maxHp,
        exp: Math.max(0, p.exp - 25),
        gold: Math.max(0, p.gold - 10)
      }));
      setTimeout(() => setGameState('menu'), 2000);
    } else {
      setCurrentQuestion(generateQuestion(enemy.difficulty));
      setFeedback('');
    }
  };

  const heal = () => {
    if (player.gold >= 30) {
      setPlayer({
        ...player,
        hp: player.maxHp,
        gold: player.gold - 30
      });
      setBattleLog(['Kamu menggunakan Potion! HP penuh.']);
    }
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-purple-900 via-blue-900 to-indigo-900 p-4">
      <div className="max-w-4xl mx-auto">
        {/* Header */}
        <div className="bg-black bg-opacity-50 rounded-lg p-4 mb-4 text-white">
          <h1 className="text-3xl font-bold text-center mb-2 flex items-center justify-center gap-2">
            <BookOpen className="text-yellow-400" />
            Algebra Quest RPG
          </h1>
          
          {/* Player Stats */}
          <div className="grid grid-cols-2 md:grid-cols-4 gap-4 mt-4">
            <div className="bg-blue-900 bg-opacity-50 rounded p-3">
              <div className="text-sm opacity-75">Level</div>
              <div className="text-2xl font-bold flex items-center gap-2">
                <Star className="text-yellow-400" size={20} />
                {player.level}
              </div>
            </div>
            <div className="bg-red-900 bg-opacity-50 rounded p-3">
              <div className="text-sm opacity-75">HP</div>
              <div className="text-2xl font-bold flex items-center gap-2">
                <Heart className="text-red-400" size={20} />
                {player.hp}/{player.maxHp}
              </div>
            </div>
            <div className="bg-orange-900 bg-opacity-50 rounded p-3">
              <div className="text-sm opacity-75">ATK/DEF</div>
              <div className="text-xl font-bold flex items-center gap-2">
                <Sword className="text-orange-400" size={18} />
                {player.attack}
                <Shield className="text-blue-400" size={18} />
                {player.defense}
              </div>
            </div>
            <div className="bg-yellow-900 bg-opacity-50 rounded p-3">
              <div className="text-sm opacity-75">Gold</div>
              <div className="text-2xl font-bold flex items-center gap-2">
                <Trophy className="text-yellow-400" size={20} />
                {player.gold}
              </div>
            </div>
          </div>
          
          {/* EXP Bar */}
          <div className="mt-3">
            <div className="flex justify-between text-sm mb-1">
              <span>EXP: {player.exp}/{player.expToNext}</span>
              <span>{Math.floor((player.exp / player.expToNext) * 100)}%</span>
            </div>
            <div className="bg-gray-700 rounded-full h-3 overflow-hidden">
              <div 
                className="bg-gradient-to-r from-green-400 to-blue-500 h-full transition-all duration-500"
                style={{ width: `${(player.exp / player.expToNext) * 100}%` }}
              />
            </div>
          </div>
        </div>

        {/* Menu Screen */}
        {gameState === 'menu' && (
          <div className="bg-black bg-opacity-50 rounded-lg p-6 text-white">
            <h2 className="text-2xl font-bold mb-4">Pilih Area Petualangan</h2>
            
            <div className="space-y-3">
              {Object.entries(stages).map(([stageNum, stageData]) => {
                const unlocked = player.level >= (parseInt(stageNum) - 1) * 4 + 1;
                return (
                  <button
                    key={stageNum}
                    onClick={() => {
                      if (unlocked) {
                        setStage(parseInt(stageNum));
                        startBattle();
                      }
                    }}
                    disabled={!unlocked}
                    className={`w-full p-4 rounded-lg text-left transition-all ${
                      unlocked 
                        ? 'bg-gradient-to-r from-purple-600 to-blue-600 hover:from-purple-500 hover:to-blue-500 cursor-pointer' 
                        : 'bg-gray-700 opacity-50 cursor-not-allowed'
                    }`}
                  >
                    <div className="flex justify-between items-center">
                      <div>
                        <div className="font-bold text-lg">{stageData.name}</div>
                        <div className="text-sm opacity-75 capitalize">
                          Difficulty: {stageData.difficulty}
                        </div>
                      </div>
                      {!unlocked && (
                        <div className="text-sm bg-red-600 px-3 py-1 rounded">
                          Unlock at Lv {(parseInt(stageNum) - 1) * 4 + 1}
                        </div>
                      )}
                    </div>
                  </button>
                );
              })}
            </div>
            
            <div className="mt-6 p-4 bg-green-900 bg-opacity-50 rounded-lg">
              <button
                onClick={heal}
                disabled={player.gold < 30 || player.hp === player.maxHp}
                className="w-full py-3 bg-green-600 hover:bg-green-500 rounded-lg font-bold disabled:opacity-50 disabled:cursor-not-allowed"
              >
                Heal (30 Gold) - HP: {player.hp}/{player.maxHp}
              </button>
            </div>
          </div>
        )}

        {/* Battle Screen */}
        {gameState === 'game' && enemy && currentQuestion && (
          <div className="space-y-4">
            {/* Enemy */}
            <div className="bg-black bg-opacity-50 rounded-lg p-6 text-white text-center">
              <h3 className="text-2xl font-bold mb-2">{enemy.name}</h3>
              <div className="mb-2">
                <div className="flex justify-between text-sm mb-1">
                  <span>HP: {enemy.hp}/{enemy.maxHp}</span>
                  <span>{Math.floor((enemy.hp / enemy.maxHp) * 100)}%</span>
                </div>
                <div className="bg-gray-700 rounded-full h-4 overflow-hidden">
                  <div 
                    className="bg-gradient-to-r from-red-500 to-orange-500 h-full transition-all duration-500"
                    style={{ width: `${(enemy.hp / enemy.maxHp) * 100}%` }}
                  />
                </div>
              </div>
            </div>

            {/* Question */}
            <div className="bg-black bg-opacity-50 rounded-lg p-6 text-white">
              <div className="flex items-center gap-2 mb-4">
                <Zap className="text-yellow-400" />
                <h3 className="text-xl font-bold">Soal Matematika:</h3>
              </div>
              <p className="text-lg mb-4 bg-blue-900 bg-opacity-30 p-4 rounded">
                {currentQuestion.question}
              </p>
              
              <div className="flex gap-2">
                <input
                  type="number"
                  step="any"
                  value={userAnswer}
                  onChange={(e) => setUserAnswer(e.target.value)}
                  onKeyPress={(e) => e.key === 'Enter' && checkAnswer()}
                  placeholder="Jawaban kamu..."
                  className="flex-1 px-4 py-3 bg-gray-800 rounded-lg text-white text-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
                />
                <button
                  onClick={checkAnswer}
                  className="px-6 py-3 bg-gradient-to-r from-green-500 to-blue-500 hover:from-green-400 hover:to-blue-400 rounded-lg font-bold"
                >
                  Jawab!
                </button>
              </div>
              
              {feedback && (
                <div className={`mt-4 p-3 rounded ${
                  feedback.startsWith('Correct') ? 'bg-green-900 bg-opacity-50' : 'bg-red-900 bg-opacity-50'
                }`}>
                  {feedback}
                </div>
              )}
            </div>

            {/* Battle Log */}
            <div className="bg-black bg-opacity-50 rounded-lg p-4 text-white">
              <h4 className="font-bold mb-2">Battle Log:</h4>
              <div className="space-y-1 max-h-32 overflow-y-auto">
                {battleLog.map((log, i) => (
                  <div key={i} className="text-sm opacity-90">{log}</div>
                ))}
              </div>
            </div>
          </div>
        )}
      </div>
    </div>
  );
};

export default AlgebraRPG;
