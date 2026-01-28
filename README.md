const express = require('express');
const app = express();
const http = require('http').createServer(app);
const io = require('socket.io')(http);

app.use(express.static(__dirname));

let players = [];
let ball = { x: 180, y: 380, dx:0, dy:0, ownerId:null, shotClock:24 };

io.on('connection', socket => {
  console.log('Player connected:', socket.id);
  players.push({id: socket.id, x:165, y:380, score:0, fouls:0});

  socket.on('update', data => {
    let p = players.find(p=>p.id===socket.id);
    if(p) Object.assign(p, data);
    socket.broadcast.emit('updatePlayers', players);
  });

  socket.on('ballUpdate', data => { Object.assign(ball, data); socket.broadcast.emit('ballUpdate', ball); });

  socket.on('disconnect', () => {
    players = players.filter(p=>p.id!==socket.id);
    console.log('Player disconnected:', socket.id);
  });
});

http.listen(3000, ()=>console.log('Server running on http://localhost:3000'));
