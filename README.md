# website-dummy

import React, { useState, useRef, useEffect } from 'react';
import { motion } from 'framer-motion';
import { BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer, PieChart, Pie, Cell } from 'recharts';
import { Play, Pause, RotateCcw, Palette, Zap, Users } from 'lucide-react';
import p5 from 'p5';

// Data mock untuk chart
const performanceData = [
  { name: 'Jan', value: 4000 },
  { name: 'Feb', value: 3000 },
  { name: 'Mar', value: 6000 },
  { name: 'Apr', value: 5000 },
  { name: 'May', value: 7500 },
  { name: 'Jun', value: 8000 },
];

const userStats = [
  { name: 'Active', value: 75, color: '#3b82f6' },
  { name: 'Inactive', value: 25, color: '#94a3b8' },
];

export default function App() {
  const [isPlaying, setIsPlaying] = useState(true);
  const [particleCount, setParticleCount] = useState(150);
  const [speed, setSpeed] = useState(1);
  const [colorMode, setColorMode] = useState('rainbow');
  const sketchRef = useRef();

  // Efek saat mount: jalankan p5 sketch
  useEffect(() => {
    if (sketchRef.current) return;

    const sketch = (p) => {
      let particles = [];
      
      p.setup = () => {
        p.createCanvas(p.windowWidth, p.windowHeight);
        particles = [];
        for (let i = 0; i < particleCount; i++) {
          particles.push(new Particle(p));
        }
      };

      p.draw = () => {
        if (!isPlaying) return;
        
        p.background(15, 23, 42, 20);
        
        // Perbarui partikel
        particles.forEach((particle, i) => {
          particle.update(speed);
          particle.show(p, colorMode);
          
          // Hubungkan partikel yang berdekatan
          for (let j = i + 1; j < particles.length; j++) {
            const d = p.dist(particle.x, particle.y, particles[j].x, particles[j].y);
            if (d < 100) {
              const alpha = p.map(d, 0, 100, 100, 0);
              p.stroke(59, 130, 246, alpha);
              p.line(particle.x, particle.y, particles[j].x, particles[j].y);
            }
          }
        });
      };

      p.windowResized = () => {
        p.resizeCanvas(p.windowWidth, p.windowHeight);
      };

      // Kelas Partikel
      class Particle {
        constructor(p) {
          this.p = p;
          this.x = p.random(p.width);
          this.y = p.random(p.height);
          this.vx = p.random(-1, 1);
          this.vy = p.random(-1, 1);
          this.size = p.random(2, 6);
          this.hue = p.random(200, 260);
        }

        update(speed) {
          this.x += this.vx * speed;
          this.y += this.vy * speed;

          // Pantul di tepi
          if (this.x < 0 || this.x > this.p.width) this.vx *= -1;
          if (this.y < 0 || this.y > this.p.height) this.vy *= -1;
        }

        show(p, mode) {
          if (mode === 'rainbow') {
            this.hue = (this.hue + 0.5) % 360;
            p.fill(this.hue, 80, 90);
          } else if (mode === 'blue') {
            p.fill(59, 130, 246);
          } else {
            p.fill(14, 165, 233);
          }
          p.noStroke();
          p.ellipse(this.x, this.y, this.size);
        }
      }
    };

    sketchRef.current = new p5(sketch, 'p5-container');

    return () => {
      if (sketchRef.current) {
        sketchRef.current.remove();
        sketchRef.current = null;
      }
    };
  }, [isPlaying, particleCount, speed, colorMode]);

  // Reset partikel
  const resetParticles = () => {
    if (sketchRef.current) {
      sketchRef.current.setup();
    }
  };

  return (
    <div className="relative min-h-screen bg-gray-900 text-white overflow-hidden">
      {/* Canvas untuk partikel */}
      <div id="p5-container" className="absolute inset-0 z-0"></div>

      {/* Overlay konten */}
      <div className="relative z-10">
        {/* Header */}
        <motion.header 
          initial={{ y: -50, opacity: 0 }}
          animate={{ y: 0, opacity: 1 }}
          className="container mx-auto px-4 py-6 flex justify-between items-center"
        >
          <motion.h1 
            whileHover={{ scale: 1.05 }}
            className="text-2xl font-bold bg-clip-text text-transparent bg-gradient-to-r from-cyan-400 to-blue-500"
          >
            Interactive Dashboard
          </motion.h1>
          <motion.button
            whileHover={{ scale: 1.05 }}
            whileTap={{ scale: 0.95 }}
            onClick={() => setIsPlaying(!isPlaying)}
            className="flex items-center gap-2 px-4 py-2 bg-blue-600 rounded-lg hover:bg-blue-700 transition-colors"
          >
            {isPlaying ? <Pause size={18} /> : <Play size={18} />}
            {isPlaying ? 'Pause' : 'Play'}
          </motion.button>
        </motion.header>

        {/* Kontrol Interaktif */}
        <motion.section 
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ delay: 0.2 }}
          className="container mx-auto px-4 py-8"
        >
          <div className="bg-gray-800/50 backdrop-blur-sm rounded-2xl p-6 border border-gray-700 max-w-4xl mx-auto">
            <h2 className="text-xl font-bold mb-4 flex items-center gap-2">
              <Zap size={20} className="text-cyan-400" />
              Particle Controls
            </h2>
            
            <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
              {/* Jumlah Partikel */}
              <div>
                <label className="block text-sm font-medium mb-2">Particle Count: {particleCount}</label>
                <input
                  type="range"
                  min="50"
                  max="300"
                  value={particleCount}
                  onChange={(e) => setParticleCount(parseInt(e.target.value))}
                  className="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer accent-cyan-500"
                />
              </div>

              {/* Kecepatan */}
              <div>
                <label className="block text-sm font-medium mb-2">Speed: {speed.toFixed(1)}x</label>
                <input
                  type="range"
                  min="0.1"
                  max="3"
                  step="0.1"
                  value={speed}
                  onChange={(e) => setSpeed(parseFloat(e.target.value))}
                  className="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer accent-blue-500"
                />
              </div>

              {/* Mode Warna */}
              <div>
                <label className="block text-sm font-medium mb-2">Color Mode</label>
                <div className="flex gap-2">
                  {[
                    { id: 'rainbow', name: 'Rainbow', bg: 'bg-gradient-to-r from-red-500 to-purple-600' },
                    { id: 'blue', name: 'Blue', bg: 'bg-blue-500' },
                    { id: 'cyan', name: 'Cyan', bg: 'bg-cyan-500' }
                  ].map((mode) => (
                    <motion.button
                      key={mode.id}
                      whileHover={{ scale: 1.05 }}
                      whileTap={{ scale: 0.95 }}
                      onClick={() => setColorMode(mode.id)}
                      className={`w-8 h-8 rounded-full ${mode.bg} ${colorMode === mode.id ? 'ring-2 ring-white' : ''}`}
                      title={mode.name}
                    />
                  ))}
                </div>
              </div>
            </div>

            <div className="mt-6 flex justify-center">
              <motion.button
                whileHover={{ scale: 1.05 }}
                whileTap={{ scale: 0.95 }}
                onClick={resetParticles}
                className="flex items-center gap-2 px-5 py-2.5 bg-gray-700 hover:bg-gray-600 rounded-lg transition-colors"
              >
                <RotateCcw size={18} />
                Reset Particles
              </motion.button>
            </div>
          </div>
        </motion.section>

        {/* Visualisasi Data */}
        <motion.section 
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ delay: 0.4 }}
          className="container mx-auto px-4 py-8"
        >
          <div className="grid grid-cols-1 lg:grid-cols-2 gap-8 max-w-6xl mx-auto">
            {/* Chart Performa */}
            <motion.div 
              whileHover={{ scale: 1.02 }}
              className="bg-gray-800/50 backdrop-blur-sm rounded-2xl p-6 border border-gray-700"
            >
              <h3 className="text-lg font-bold mb-4 flex items-center gap-2">
                <Palette size={20} className="text-cyan-400" />
                Performance Analytics
              </h3>
              <div className="h-80">
                <ResponsiveContainer width="100%" height="100%">
                  <BarChart data={performanceData}>
                    <CartesianGrid strokeDasharray="3 3" stroke="#334155" />
                    <XAxis dataKey="name" stroke="#94a3b8" />
                    <YAxis stroke="#94a3b8" />
                    <Tooltip 
                      contentStyle={{ 
                        backgroundColor: '#1e293b', 
                        borderColor: '#334155',
                        borderRadius: '0.5rem'
                      }} 
                    />
                    <Bar 
                      dataKey="value" 
                      fill="#3b82f6" 
                      radius={[4, 4, 0, 0]}
                    />
                  </BarChart>
                </ResponsiveContainer>
              </div>
            </motion.div>

            {/* Statistik Pengguna */}
            <motion.div 
              whileHover={{ scale: 1.02 }}
              className="bg-gray-800/50 backdrop-blur-sm rounded-2xl p-6 border border-gray-700 flex flex-col"
            >
              <h3 className="text-lg font-bold mb-4 flex items-center gap-2">
                <Users size={20} className="text-cyan-400" />
                User Statistics
              </h3>
              <div className="flex-1 flex items-center justify-center">
                <div className="w-64 h-64">
                  <ResponsiveContainer width="100%" height="100%">
                    <PieChart>
                      <Pie
                        data={userStats}
                        cx="50%"
                        cy="50%"
                        innerRadius={60}
                        outerRadius={80}
                        paddingAngle={5}
                        dataKey="value"
                      >
                        {userStats.map((entry, index) => (
                          <Cell key={`cell-${index}`} fill={entry.color} />
                        ))}
                      </Pie>
                      <Tooltip 
                        formatter={(value) => [`${value}%`, 'Percentage']}
                        contentStyle={{ 
                          backgroundColor: '#1e293b', 
                          borderColor: '#334155',
                          borderRadius: '0.5rem'
                        }} 
                      />
                    </PieChart>
                  </ResponsiveContainer>
                </div>
              </div>
              <div className="mt-4 grid grid-cols-2 gap-4">
                {userStats.map((stat) => (
                  <div key={stat.name} className="flex items-center gap-2">
                    <div 
                      className="w-4 h-4 rounded-full" 
                      style={{ backgroundColor: stat.color }}
                    ></div>
                    <span>{stat.name}: {stat.value}%</span>
                  </div>
                ))}
              </div>
            </motion.div>
          </div>
        </motion.section>

        {/* Footer */}
        <motion.footer 
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          transition={{ delay: 0.6 }}
          className="container mx-auto px-4 py-8 text-center text-gray-500 text-sm"
        >
          <p>✨ Interactive Dashboard with p5.js & Recharts</p>
          <p className="mt-1">Deploy to Vercel/Netlify for global access</p>
        </motion.footer>
      </div>
    </div>
  );
}
