import React, { useState, useEffect, useRef } from 'react';
import { Target, Heart, Skull, Zap } from 'lucide-react';

const IOShooterGame = () => {
  const canvasRef = useRef(null);
  const [gameState, setGameState] = useState('menu');
  const [score, setScore] = useState(0);
  const [health, setHealth] = useState(100);
  const [kills, setKills] = useState(0);
  const [currentWeapon, setCurrentWeapon] = useState(0);
  const [playerName, setPlayerName] = useState('');
  const [displayName, setDisplayName] = useState('');
  const [highScore, setHighScore] = useState(0);
  const [highScoreName, setHighScoreName] = useState('');

  // 載入最高分
  useEffect(() => {
    const savedHighScore = localStorage.getItem('shooterIOHighScore');
    const savedHighScoreName = localStorage.getItem('shooterIOHighScoreName');
    if (savedHighScore) setHighScore(parseInt(savedHighScore));
    if (savedHighScoreName) setHighScoreName(savedHighScoreName);
  }, []);
  
  const weapons = [
    { name: '手槍', cooldown: 300, damage: 1, bulletSpeed: 8, bulletSize: 4, color: '#feca57', burst: 1 },
    { name: '衝鋒槍', cooldown: 100, damage: 1, bulletSpeed: 9, bulletSize: 3, color: '#ff6b6b', burst: 1 },
    { name: '霰彈槍', cooldown: 800, damage: 1, bulletSpeed: 7, bulletSize: 5, color: '#ff9ff3', burst: 5 },
    { name: '狙擊槍', cooldown: 1200, damage: 3, bulletSpeed: 15, bulletSize: 6, color: '#54a0ff', burst: 1 },
    { name: '火箭筒', cooldown: 2000, damage: 5, bulletSpeed: 5, bulletSize: 10, color: '#ff6348', burst: 1, explosive: true }
  ];
  
  const gameRef = useRef({
    player: { x: 400, y: 300, angle: 0, speed: 3, size: 20 },
    bullets: [],
    enemyBullets: [],
    enemies: [],
    particles: [],
    healthPacks: [],
    keys: {},
    mouseX: 400,
    mouseY: 300,
    mouseDown: false,
    lastShot: 0,
    enemySpawnTimer: 0,
    healthPackTimer: 0,
    animationId: null,
    currentWeapon: 0
  });

  useEffect(() => {
    if (gameState !== 'playing') return;

    const canvas = canvasRef.current;
    const ctx = canvas.getContext('2d');
    const game = gameRef.current;

    const handleKeyDown = (e) => {
      game.keys[e.key.toLowerCase()] = true;
      
      // 切換武器 (1-5)
      if (e.key >= '1' && e.key <= '5') {
        const weaponIndex = parseInt(e.key) - 1;
        game.currentWeapon = weaponIndex;
        setCurrentWeapon(weaponIndex);
      }
    };
    
    const handleKeyUp = (e) => {
      game.keys[e.key.toLowerCase()] = false;
    };

    const handleMouseMove = (e) => {
      const rect = canvas.getBoundingClientRect();
      game.mouseX = e.clientX - rect.left;
      game.mouseY = e.clientY - rect.top;
    };

    const handleMouseDown = () => {
      game.mouseDown = true;
    };

    const handleMouseUp = () => {
      game.mouseDown = false;
    };

    const shoot = () => {
      const now = Date.now();
      const weapon = weapons[game.currentWeapon];
      
      if (now - game.lastShot < weapon.cooldown) return;
      
      game.lastShot = now;
      const angle = game.player.angle;
      
      // 霰彈槍發射多發子彈
      if (weapon.burst > 1) {
        for (let i = 0; i < weapon.burst; i++) {
          const spreadAngle = angle + (Math.random() - 0.5) * 0.5;
          game.bullets.push({
            x: game.player.x,
            y: game.player.y,
            vx: Math.cos(spreadAngle) * weapon.bulletSpeed,
            vy: Math.sin(spreadAngle) * weapon.bulletSpeed,
            size: weapon.bulletSize,
            damage: weapon.damage,
            color: weapon.color,
            explosive: weapon.explosive || false
          });
        }
      } else {
        game.bullets.push({
          x: game.player.x,
          y: game.player.y,
          vx: Math.cos(angle) * weapon.bulletSpeed,
          vy: Math.sin(angle) * weapon.bulletSpeed,
          size: weapon.bulletSize,
          damage: weapon.damage,
          color: weapon.color,
          explosive: weapon.explosive || false
        });
      }
    };

    const spawnEnemy = () => {
      const side = Math.floor(Math.random() * 4);
      let x, y;
      
      if (side === 0) { x = Math.random() * 800; y = -20; }
      else if (side === 1) { x = 820; y = Math.random() * 600; }
      else if (side === 2) { x = Math.random() * 800; y = 620; }
      else { x = -20; y = Math.random() * 600; }
      
      game.enemies.push({
        x, y,
        size: 18,
        health: 3,
        maxHealth: 3,
        speed: 1 + Math.random() * 0.5,
        lastShot: 0,
        shootCooldown: 1500 + Math.random() * 1000,
        burstCount: 0,
        burstMax: 2 + Math.floor(Math.random() * 3),
        burstDelay: 150
      });
    };

    const spawnHealthPack = () => {
      game.healthPacks.push({
        x: 100 + Math.random() * 600,
        y: 100 + Math.random() * 400,
        size: 15,
        healAmount: 30,
        pulsePhase: 0
      });
    };

    const createParticles = (x, y, color, count = 8) => {
      for (let i = 0; i < count; i++) {
        const angle = (Math.PI * 2 * i) / count;
        game.particles.push({
          x, y,
          vx: Math.cos(angle) * (2 + Math.random() * 2),
          vy: Math.sin(angle) * (2 + Math.random() * 2),
          life: 30,
          color
        });
      }
    };

    const createExplosion = (x, y) => {
      for (let i = 0; i < 20; i++) {
        const angle = (Math.PI * 2 * i) / 20;
        game.particles.push({
          x, y,
          vx: Math.cos(angle) * (3 + Math.random() * 4),
          vy: Math.sin(angle) * (3 + Math.random() * 4),
          life: 40,
          color: ['#ff6348', '#ff9ff3', '#feca57'][Math.floor(Math.random() * 3)]
        });
      }
    };

    const gameLoop = () => {
      ctx.fillStyle = '#1a1a2e';
      ctx.fillRect(0, 0, 800, 600);

      // 網格背景
      ctx.strokeStyle = '#16213e33';
      ctx.lineWidth = 1;
      for (let i = 0; i < 800; i += 40) {
        ctx.beginPath();
        ctx.moveTo(i, 0);
        ctx.lineTo(i, 600);
        ctx.stroke();
      }
      for (let i = 0; i < 600; i += 40) {
        ctx.beginPath();
        ctx.moveTo(0, i);
        ctx.lineTo(800, i);
        ctx.stroke();
      }

      // 玩家移動
      if (game.keys['w'] || game.keys['arrowup']) game.player.y -= game.player.speed;
      if (game.keys['s'] || game.keys['arrowdown']) game.player.y += game.player.speed;
      if (game.keys['a'] || game.keys['arrowleft']) game.player.x -= game.player.speed;
      if (game.keys['d'] || game.keys['arrowright']) game.player.x += game.player.speed;

      // 邊界限制
      game.player.x = Math.max(20, Math.min(780, game.player.x));
      game.player.y = Math.max(20, Math.min(580, game.player.y));

      // 計算玩家角度
      const dx = game.mouseX - game.player.x;
      const dy = game.mouseY - game.player.y;
      game.player.angle = Math.atan2(dy, dx);

      // 連發射擊
      if (game.mouseDown) {
        shoot();
      }

      // 更新子彈
      game.bullets = game.bullets.filter(bullet => {
        bullet.x += bullet.vx;
        bullet.y += bullet.vy;
        return bullet.x > 0 && bullet.x < 800 && bullet.y > 0 && bullet.y < 600;
      });

      // 更新敵人子彈
      game.enemyBullets = game.enemyBullets.filter(bullet => {
        bullet.x += bullet.vx;
        bullet.y += bullet.vy;
        return bullet.x > 0 && bullet.x < 800 && bullet.y > 0 && bullet.y < 600;
      });

      // 生成敵人
      game.enemySpawnTimer++;
      if (game.enemySpawnTimer > 100) {
        spawnEnemy();
        game.enemySpawnTimer = 0;
      }

      // 生成補血包
      game.healthPackTimer++;
      if (game.healthPackTimer > 600) { // 每10秒生成一個
        spawnHealthPack();
        game.healthPackTimer = 0;
      }

      // 更新敵人
      const now = Date.now();
      game.enemies.forEach(enemy => {
        const dx = game.player.x - enemy.x;
        const dy = game.player.y - enemy.y;
        const dist = Math.sqrt(dx * dx + dy * dy);
        
        // 如果距離夠遠就接近玩家，如果太近就停下來射擊
        if (dist > 150) {
          enemy.x += (dx / dist) * enemy.speed;
          enemy.y += (dy / dist) * enemy.speed;
        }
        
        // 敵人連發射擊
        if (enemy.burstCount > 0) {
          if (now - enemy.lastShot > enemy.burstDelay) {
            enemy.lastShot = now;
            enemy.burstCount--;
            const angle = Math.atan2(dy, dx);
            game.enemyBullets.push({
              x: enemy.x,
              y: enemy.y,
              vx: Math.cos(angle) * 5,
              vy: Math.sin(angle) * 5,
              size: 5,
              damage: 5
            });
          }
        } else if (now - enemy.lastShot > enemy.shootCooldown) {
          enemy.lastShot = now;
          enemy.burstCount = enemy.burstMax;
        }
      });

      // 碰撞檢測：子彈 vs 敵人
      game.bullets = game.bullets.filter(bullet => {
        let hit = false;
        
        // 爆炸傷害範圍
        if (bullet.explosive) {
          game.enemies.forEach(enemy => {
            const dx = bullet.x - enemy.x;
            const dy = bullet.y - enemy.y;
            const dist = Math.sqrt(dx * dx + dy * dy);
            
            if (dist < 80) {
              enemy.health -= bullet.damage;
            }
          });
        }
        
        game.enemies = game.enemies.filter(enemy => {
          const dx = bullet.x - enemy.x;
          const dy = bullet.y - enemy.y;
          const dist = Math.sqrt(dx * dx + dy * dy);
          
          if (dist < enemy.size) {
            enemy.health -= bullet.damage;
            hit = true;
            
            if (bullet.explosive) {
              createExplosion(bullet.x, bullet.y);
            }
            
            if (enemy.health <= 0) {
              createParticles(enemy.x, enemy.y, '#ff6b6b', 12);
              setScore(s => s + 100);
              setKills(k => k + 1);
              return false;
            }
          }
          return true;
        });
        
        if (hit && bullet.explosive) {
          return false;
        }
        
        return !hit;
      });

      // 碰撞檢測：玩家 vs 敵人
      game.enemies = game.enemies.filter(enemy => {
        const dx = game.player.x - enemy.x;
        const dy = game.player.y - enemy.y;
        const dist = Math.sqrt(dx * dx + dy * dy);
        
        if (dist < game.player.size + enemy.size) {
          setHealth(h => {
            const newHealth = h - 10;
            if (newHealth <= 0) {
              setGameState('gameover');
            }
            return Math.max(0, newHealth);
          });
          createParticles(enemy.x, enemy.y, '#ff6b6b');
          return false;
        }
        return true;
      });

      // 碰撞檢測：敵人子彈 vs 玩家
      game.enemyBullets = game.enemyBullets.filter(bullet => {
        const dx = game.player.x - bullet.x;
        const dy = game.player.y - bullet.y;
        const dist = Math.sqrt(dx * dx + dy * dy);
        
        if (dist < game.player.size) {
          setHealth(h => {
            const newHealth = h - bullet.damage;
            if (newHealth <= 0) {
              setGameState('gameover');
            }
            return Math.max(0, newHealth);
          });
          createParticles(bullet.x, bullet.y, '#48dbfb');
          return false;
        }
        return true;
      });

      // 更新補血包
      game.healthPacks.forEach(pack => {
        pack.pulsePhase += 0.1;
      });

      // 碰撞檢測：玩家 vs 補血包
      game.healthPacks = game.healthPacks.filter(pack => {
        const dx = game.player.x - pack.x;
        const dy = game.player.y - pack.y;
        const dist = Math.sqrt(dx * dx + dy * dy);
        
        if (dist < game.player.size + pack.size) {
          setHealth(h => Math.min(100, h + pack.healAmount));
          createParticles(pack.x, pack.y, '#2ecc71', 12);
          setScore(s => s + 50);
          return false;
        }
        return true;
      });

      // 更新粒子
      game.particles = game.particles.filter(p => {
        p.x += p.vx;
        p.y += p.vy;
        p.life--;
        p.vx *= 0.95;
        p.vy *= 0.95;
        return p.life > 0;
      });

      // 繪製粒子
      game.particles.forEach(p => {
        ctx.fillStyle = p.color + Math.floor((p.life / 40) * 255).toString(16).padStart(2, '0');
        ctx.fillRect(p.x - 2, p.y - 2, 4, 4);
      });

      // 繪製敵人
      game.enemies.forEach(enemy => {
        // 計算敵人面向玩家的角度
        const dx = game.player.x - enemy.x;
        const dy = game.player.y - enemy.y;
        const angle = Math.atan2(dy, dx);
        
        ctx.save();
        ctx.translate(enemy.x, enemy.y);
        ctx.rotate(angle);
        
        // 敵人身體
        ctx.fillStyle = '#ff6b6b';
        ctx.beginPath();
        ctx.arc(0, 0, enemy.size, 0, Math.PI * 2);
        ctx.fill();
        
        ctx.strokeStyle = '#ffffff';
        ctx.lineWidth = 2;
        ctx.stroke();
        
        // 敵人槍管
        ctx.fillStyle = '#8B0000';
        ctx.fillRect(10, -2, 12, 4);
        
        ctx.restore();
        
        // 血量條
        ctx.fillStyle = '#333333';
        ctx.fillRect(enemy.x - 15, enemy.y - 25, 30, 4);
        ctx.fillStyle = '#ff6b6b';
        ctx.fillRect(enemy.x - 15, enemy.y - 25, (enemy.health / enemy.maxHealth) * 30, 4);
      });

      // 繪製子彈
      game.bullets.forEach(bullet => {
        ctx.fillStyle = bullet.color;
        ctx.beginPath();
        ctx.arc(bullet.x, bullet.y, bullet.size, 0, Math.PI * 2);
        ctx.fill();
        
        // 爆炸子彈的光暈
        if (bullet.explosive) {
          ctx.fillStyle = bullet.color + '44';
          ctx.beginPath();
          ctx.arc(bullet.x, bullet.y, bullet.size * 2, 0, Math.PI * 2);
          ctx.fill();
        }
      });

      // 繪製敵人子彈
      game.enemyBullets.forEach(bullet => {
        ctx.fillStyle = '#ff4757';
        ctx.beginPath();
        ctx.arc(bullet.x, bullet.y, bullet.size, 0, Math.PI * 2);
        ctx.fill();
        
        // 紅色光暈
        ctx.fillStyle = '#ff475744';
        ctx.beginPath();
        ctx.arc(bullet.x, bullet.y, bullet.size * 1.5, 0, Math.PI * 2);
        ctx.fill();
      });

      // 繪製補血包
      game.healthPacks.forEach(pack => {
        const pulseSize = pack.size + Math.sin(pack.pulsePhase) * 3;
        
        // 外層光暈
        ctx.fillStyle = '#2ecc7144';
        ctx.beginPath();
        ctx.arc(pack.x, pack.y, pulseSize * 1.8, 0, Math.PI * 2);
        ctx.fill();
        
        // 補血包主體
        ctx.fillStyle = '#2ecc71';
        ctx.fillRect(pack.x - pulseSize, pack.y - 3, pulseSize * 2, 6);
        ctx.fillRect(pack.x - 3, pack.y - pulseSize, 6, pulseSize * 2);
        
        // 白色邊框
        ctx.strokeStyle = '#ffffff';
        ctx.lineWidth = 2;
        ctx.strokeRect(pack.x - pulseSize, pack.y - 3, pulseSize * 2, 6);
        ctx.strokeRect(pack.x - 3, pack.y - pulseSize, 6, pulseSize * 2);
      });

      // 繪製玩家
      ctx.save();
      ctx.translate(game.player.x, game.player.y);
      ctx.rotate(game.player.angle);
      
      ctx.fillStyle = '#48dbfb';
      ctx.beginPath();
      ctx.arc(0, 0, game.player.size, 0, Math.PI * 2);
      ctx.fill();
      
      ctx.strokeStyle = '#ffffff';
      ctx.lineWidth = 2;
      ctx.stroke();
      
      // 不同武器的槍管
      const weapon = weapons[game.currentWeapon];
      ctx.fillStyle = '#2c3e50';
      if (game.currentWeapon === 3) { // 狙擊槍
        ctx.fillRect(10, -2, 20, 4);
      } else if (game.currentWeapon === 4) { // 火箭筒
        ctx.fillRect(10, -4, 18, 8);
      } else {
        ctx.fillRect(10, -3, 15, 6);
      }
      
      ctx.restore();

      game.animationId = requestAnimationFrame(gameLoop);
    };

    window.addEventListener('keydown', handleKeyDown);
    window.addEventListener('keyup', handleKeyUp);
    canvas.addEventListener('mousemove', handleMouseMove);
    canvas.addEventListener('mousedown', handleMouseDown);
    canvas.addEventListener('mouseup', handleMouseUp);

    gameLoop();

    return () => {
      window.removeEventListener('keydown', handleKeyDown);
      window.removeEventListener('keyup', handleKeyUp);
      canvas.removeEventListener('mousemove', handleMouseMove);
      canvas.removeEventListener('mousedown', handleMouseDown);
      canvas.removeEventListener('mouseup', handleMouseUp);
      if (game.animationId) {
        cancelAnimationFrame(game.animationId);
      }
    };
  }, [gameState, currentWeapon]);

  const startGame = () => {
    if (!playerName.trim()) {
      alert('請輸入玩家名稱！');
      return;
    }
    setDisplayName(playerName);
    setGameState('playing');
    setScore(0);
    setHealth(100);
    setKills(0);
    setCurrentWeapon(0);
    gameRef.current = {
      player: { x: 400, y: 300, angle: 0, speed: 3, size: 20 },
      bullets: [],
      enemyBullets: [],
      enemies: [],
      particles: [],
      healthPacks: [],
      keys: {},
      mouseX: 400,
      mouseY: 300,
      mouseDown: false,
      lastShot: 0,
      enemySpawnTimer: 0,
      healthPackTimer: 0,
      animationId: null,
      currentWeapon: 0
    };
  };

  // 遊戲結束時檢查並更新最高分
  useEffect(() => {
    if (gameState === 'gameover' && score > highScore) {
      setHighScore(score);
      setHighScoreName(displayName);
      localStorage.setItem('shooterIOHighScore', score.toString());
      localStorage.setItem('shooterIOHighScoreName', displayName);
    }
  }, [gameState, score, highScore, displayName]);

  return (
    <div className="flex flex-col items-center justify-center min-h-screen bg-gradient-to-br from-slate-900 via-purple-900 to-slate-900 p-4">
      {gameState === 'menu' && (
        <div className="text-center">
          <h1 className="text-6xl font-bold text-white mb-4 drop-shadow-lg">
            <Target className="inline-block mr-3" size={60} />
            SHOOTER.IO
          </h1>
          <p className="text-xl text-gray-300 mb-8">多武器俯視角射擊遊戲</p>
          
          {/* 最高分顯示 */}
          {highScore > 0 && (
            <div className="bg-gradient-to-r from-yellow-500 to-orange-500 p-4 rounded-lg mb-6 max-w-md mx-auto">
              <p className="text-white font-bold text-xl">🏆 最高分記錄</p>
              <p className="text-white text-2xl">{highScoreName}: {highScore} 分</p>
            </div>
          )}

          {/* 玩家名稱輸入 */}
          <div className="mb-6">
            <input
              type="text"
              value={playerName}
              onChange={(e) => setPlayerName(e.target.value)}
              onKeyPress={(e) => e.key === 'Enter' && startGame()}
              placeholder="輸入你的名稱..."
              maxLength={15}
              className="px-6 py-3 text-xl rounded-lg bg-slate-800 text-white border-2 border-purple-500 focus:border-pink-500 outline-none w-64"
            />
          </div>

          <div className="bg-slate-800 bg-opacity-80 p-6 rounded-lg mb-6 text-left max-w-md mx-auto">
            <h3 className="text-white font-bold mb-3">遊戲說明：</h3>
            <p className="text-gray-300 mb-2">🎮 WASD 或方向鍵移動</p>
            <p className="text-gray-300 mb-2">🖱️ 滑鼠移動瞄準</p>
            <p className="text-gray-300 mb-2">🔫 按住滑鼠連發射擊</p>
            <p className="text-gray-300 mb-2">🔢 按 1-5 切換武器</p>
            <p className="text-gray-300 mb-2">⚠️ 小心敵人的連發攻擊！</p>
            <p className="text-gray-300 mb-2">💚 拾取綠色十字補血包</p>
            <p className="text-gray-300">💀 消滅敵人，生存下去！</p>
            <div className="mt-4 pt-4 border-t border-gray-600">
              <p className="text-yellow-400 text-sm mb-2">武器列表：</p>
              <p className="text-gray-400 text-xs">1️⃣ 手槍 - 平衡</p>
              <p className="text-gray-400 text-xs">2️⃣ 衝鋒槍 - 快速連發</p>
              <p className="text-gray-400 text-xs">3️⃣ 霰彈槍 - 散彈攻擊</p>
              <p className="text-gray-400 text-xs">4️⃣ 狙擊槍 - 高傷害</p>
              <p className="text-gray-400 text-xs">5️⃣ 火箭筒 - 爆炸範圍</p>
            </div>
          </div>
          <button
            onClick={startGame}
            className="bg-gradient-to-r from-purple-500 to-pink-500 text-white px-12 py-4 rounded-full text-2xl font-bold hover:scale-110 transform transition shadow-2xl"
          >
            開始遊戲
          </button>
        </div>
      )}

      {gameState === 'playing' && (
        <div className="relative">
          <div className="absolute top-0 left-0 right-0 flex justify-between p-4 text-white z-10">
            <div className="bg-slate-800 bg-opacity-80 px-4 py-2 rounded-lg">
              <p className="text-xs text-gray-400">玩家</p>
              <p className="font-bold">{displayName}</p>
            </div>
            <div className="bg-slate-800 bg-opacity-80 px-4 py-2 rounded-lg flex items-center">
              <Target className="mr-2" size={20} />
              <span className="font-bold">分數: {score}</span>
            </div>
            <div className="bg-slate-800 bg-opacity-80 px-4 py-2 rounded-lg flex items-center">
              <Skull className="mr-2" size={20} />
              <span className="font-bold">擊殺: {kills}</span>
            </div>
            <div className="bg-slate-800 bg-opacity-80 px-4 py-2 rounded-lg flex items-center">
              <Heart className="mr-2" size={20} fill={health > 30 ? '#ff6b6b' : '#ffffff'} />
              <div className="w-32 h-4 bg-gray-700 rounded-full overflow-hidden">
                <div 
                  className="h-full bg-gradient-to-r from-red-500 to-pink-500 transition-all"
                  style={{ width: `${health}%` }}
                />
              </div>
            </div>
          </div>
          
          {/* 武器選擇器 */}
          <div className="absolute bottom-4 left-1/2 transform -translate-x-1/2 flex gap-2 z-10">
            {weapons.map((weapon, index) => (
              <div
                key={index}
                className={`px-3 py-2 rounded-lg font-bold text-sm transition cursor-pointer ${
                  currentWeapon === index
                    ? 'bg-yellow-500 text-black scale-110'
                    : 'bg-slate-800 bg-opacity-80 text-white hover:bg-slate-700'
                }`}
                onClick={() => {
                  gameRef.current.currentWeapon = index;
                  setCurrentWeapon(index);
                }}
              >
                <div className="flex items-center gap-1">
                  <span>{index + 1}</span>
                  <Zap size={14} />
                </div>
                <div className="text-xs">{weapon.name}</div>
              </div>
            ))}
          </div>
          
          <canvas
            ref={canvasRef}
            width={800}
            height={600}
            className="border-4 border-purple-500 rounded-lg shadow-2xl cursor-crosshair"
          />
        </div>
      )}

      {gameState === 'gameover' && (
        <div className="text-center">
          <h1 className="text-6xl font-bold text-red-500 mb-4 drop-shadow-lg">
            遊戲結束
          </h1>
          <div className="bg-slate-800 bg-opacity-80 p-8 rounded-lg mb-6">
            <p className="text-white text-2xl mb-2">玩家: {displayName}</p>
            <p className="text-white text-3xl mb-4">最終分數: {score}</p>
            <p className="text-gray-300 text-2xl mb-4">擊殺數: {kills}</p>
            
            {score === highScore && score > 0 && (
              <div className="mt-4 p-4 bg-gradient-to-r from-yellow-500 to-orange-500 rounded-lg">
                <p className="text-white text-2xl font-bold">🎉 新記錄！🎉</p>
                <p className="text-white">恭喜你打破最高分！</p>
              </div>
            )}
            
            {highScore > 0 && score !== highScore && (
              <div className="mt-4 p-3 bg-slate-700 rounded-lg">
                <p className="text-gray-300 text-sm">目前最高分</p>
                <p className="text-yellow-400 text-xl">{highScoreName}: {highScore} 分</p>
              </div>
            )}
          </div>
          <button
            onClick={startGame}
            className="bg-gradient-to-r from-green-500 to-blue-500 text-white px-12 py-4 rounded-full text-2xl font-bold hover:scale-110 transform transition shadow-2xl"
          >
            再玩一次
          </button>
        </div>
      )}
    </div>
  );
};

export default IOShooterGame;
