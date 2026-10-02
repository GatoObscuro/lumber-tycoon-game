const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');

const moneyValue = document.getElementById('moneyValue');
const woodValue = document.getElementById('woodValue');
const carValue = document.getElementById('carValue');
const axeShop = document.getElementById('axeShop');
const upgradeShop = document.getElementById('upgradeShop');
const currentAxeName = document.getElementById('currentAxeName');
const speedValue = document.getElementById('speedValue');
const capacityValue = document.getElementById('capacityValue');

const world = {
  width: canvas.width,
  height: canvas.height,
  groundY: 540,
};

const treeTypes = {
  pine: {
    name: 'Pinheiro',
    trunkColor: '#7b4d29',
    leafColor: '#32c77c',
    trunkWidth: 10,
    segmentLength: 26,
    maxDepth: 3,
    branchChance: 0.7,
    value: 10,
  },
  oak: {
    name: 'Carvalho',
    trunkColor: '#8b5a2b',
    leafColor: '#4ba65a',
    trunkWidth: 12,
    segmentLength: 28,
    maxDepth: 4,
    branchChance: 0.72,
    value: 14,
  },
  mahogany: {
    name: 'Mogno',
    trunkColor: '#7b352d',
    leafColor: '#d97750',
    trunkWidth: 13,
    segmentLength: 30,
    maxDepth: 4,
    branchChance: 0.7,
    value: 20,
  },
  ancient: {
    name: 'Árvore Ancestral',
    trunkColor: '#4a3a63',
    leafColor: '#8b5cf6',
    trunkWidth: 15,
    segmentLength: 32,
    maxDepth: 5,
    branchChance: 0.8,
    value: 28,
  },
};

const axes = [
  { id: 'hand', name: 'Machado de Mão', cost: 0, damage: 1, reach: 42, speed: 3.4 },
  { id: 'iron', name: 'Machado de Ferro', cost: 45, damage: 2, reach: 52, speed: 3.7 },
  { id: 'steel', name: 'Machado de Aço', cost: 110, damage: 3, reach: 64, speed: 4.0 },
  { id: 'titanium', name: 'Machado de Titânio', cost: 230, damage: 4, reach: 76, speed: 4.3 },
  { id: 'legendary', name: 'Machado Lendário', cost: 450, damage: 6, reach: 92, speed: 4.7 },
];

const upgrades = {
  cart: { name: 'Carro de Transporte', cost: 120 },
  cart2: { name: 'Carro Melhorado', cost: 260 },
  sawmill: { name: 'Serraria', cost: 180 },
};

const controls = {};

const state = {
  money: 30,
  wood: 0,
  inventory: 0,
  axeIndex: 0,
  cartLevel: 0,
  sawmillLevel: 0,
  trees: [],
  player: {
    x: 120,
    y: 470,
    radius: 15,
    speed: axes[0].speed,
  },
  hoverFlash: 0,
  saleCooldown: 0,
};

function clamp(value, min, max) {
  return Math.min(max, Math.max(min, value));
}

function randomBetween(min, max) {
  return Math.random() * (max - min) + min;
}

function getCurrentAxe() {
  return axes[state.axeIndex];
}

function getCapacity() {
  return 10 + state.cartLevel * 8 + state.sawmillLevel * 4;
}

function buildTree(x, kind) {
  const type = treeTypes[kind];
  const root = {
    x,
    y: world.groundY,
    parent: null,
    children: [],
    thickness: type.trunkWidth,
    alive: true,
    connected: true,
    vx: 0,
    vy: 0,
    isRoot: true,
  };

  const nodes = [root];

  function addBranch(parent, depth, angle, length) {
    if (depth > type.maxDepth) return;

    const child = {
      x: parent.x + Math.cos(angle) * length,
      y: parent.y + Math.sin(angle) * length,
      parent,
      children: [],
      thickness: Math.max(2, parent.thickness * 0.8),
      alive: true,
      connected: true,
      vx: 0,
      vy: 0,
      isRoot: false,
    };

    nodes.push(child);
    parent.children.push(child);

    const nextDepth = depth + 1;
    if (nextDepth <= type.maxDepth && Math.random() < type.branchChance) {
      const leftAngle = angle - randomBetween(0.4, 1.1);
      const rightAngle = angle + randomBetween(0.4, 1.1);
      const lengthFactor = randomBetween(0.66, 0.85);
      addBranch(child, nextDepth, leftAngle, length * lengthFactor);
      if (Math.random() < 0.6) {
        addBranch(child, nextDepth, rightAngle, length * lengthFactor);
      }
    }
  }

  const trunkCount = 5 + Math.floor(Math.random() * 3);
  let trunk = root;
  for (let i = 0; i < trunkCount; i++) {
    const length = type.segmentLength * (0.9 + i * 0.18);
    const angle = -Math.PI / 2 + randomBetween(-0.16, 0.16);
    const child = {
      x: trunk.x + Math.cos(angle) * length,
      y: trunk.y + Math.sin(angle) * length,
      parent: trunk,
      children: [],
      thickness: Math.max(4, trunk.thickness * 0.8),
      alive: true,
      connected: true,
      vx: 0,
      vy: 0,
      isRoot: false,
    };
    trunk.children.push(child);
    nodes.push(child);
    trunk = child;
  }

  if (trunk && trunk.parent) {
    addBranch(trunk, 1, -Math.PI / 2 - randomBetween(0.5, 1.1), type.segmentLength * 0.75);
    addBranch(trunk, 1, -Math.PI / 2 + randomBetween(0.5, 1.1), type.segmentLength * 0.75);
  }

  return { kind, type, nodes, root, cut: false };
}

function seedWorld() {
  state.trees = [];
  const kinds = Object.keys(treeTypes);
  for (let i = 0; i < 12; i++) {
    const kind = kinds[Math.floor(Math.random() * kinds.length)];
    const x = 80 + i * 70 + randomBetween(0, 30);
    state.trees.push(buildTree(x, kind));
  }
}

function drawGround() {
  ctx.fillStyle = '#5aa95d';
  ctx.fillRect(0, world.groundY, world.width, world.height - world.groundY);

  ctx.fillStyle = '#7c5e3a';
  ctx.fillRect(0, world.groundY + 18, world.width, 16);

  ctx.fillStyle = '#427d34';
  ctx.fillRect(0, world.groundY - 18, world.width, 18);
}

function renderTree(tree) {
  for (const node of tree.nodes) {
    if (!node.parent || !node.alive || !node.parent.alive || !node.connected || !node.parent.connected) continue;
    ctx.beginPath();
    ctx.moveTo(node.parent.x, node.parent.y);
    ctx.lineTo(node.x, node.y);
    ctx.lineWidth = node.thickness;
    ctx.strokeStyle = tree.type.trunkColor;
    ctx.stroke();
  }

  for (const node of tree.nodes) {
    if (!node.alive || !node.connected) continue;
    if (node.isRoot) continue;
    const isLeaf = node.children.length === 0;
    if (isLeaf || Math.random() > 0.5) {
      ctx.beginPath();
      ctx.fillStyle = tree.type.leafColor;
      ctx.arc(node.x, node.y, 9 + (isLeaf ? 2 : 0), 0, Math.PI * 2);
      ctx.fill();
    }
  }
}

function renderTrees() {
  for (const tree of state.trees) {
    renderTree(tree);
  }
}

function updateTreePhysics(tree) {
  for (const node of tree.nodes) {
    if (!node.alive || node.connected) continue;
    node.vy += 0.18;
    node.x += node.vx;
    node.y += node.vy;
    node.vx *= 0.99;

    if (node.y > world.groundY) {
      node.y = world.groundY;
      node.vy *= -0.16;
      node.vx *= 0.7;
    }

    if (node.x < 0 || node.x > world.width) {
      node.x = clamp(node.x, 0, world.width);
      node.vx *= -0.55;
    }
  }
}

function detachSubtree(node) {
  const stack = [node];
  while (stack.length) {
    const current = stack.pop();
    if (!current) continue;
    current.connected = false;
    current.vx = randomBetween(-2.4, 2.4);
    current.vy = randomBetween(-1.5, 0.3);
    for (const child of current.children) {
      stack.push(child);
    }
  }
}

function chopNearestBranch(x, y) {
  let bestNode = null;
  let bestDist = Infinity;

  for (const tree of state.trees) {
    if (tree.cut) continue;
    for (const node of tree.nodes) {
      if (!node.alive || node.isRoot || !node.connected) continue;
      const dist = Math.hypot(node.x - x, node.y - y);
      if (dist < getCurrentAxe().reach && dist < bestDist) {
        bestDist = dist;
        bestNode = { tree, node };
      }
    }
  }

  if (!bestNode) return false;

  const { tree, node } = bestNode;
  if (node.thickness <= getCurrentAxe().damage || Math.random() < 0.6) {
    detachSubtree(node);
    tree.cut = true;
    state.wood += 1;
    state.inventory = clamp(state.inventory + 1, 0, getCapacity());
    state.hoverFlash = 10;
    return true;
  }

  node.thickness = Math.max(1, node.thickness - 0.7);
  state.hoverFlash = 6;
  return true;
}

function drawPlayer() {
  const { x, y, radius } = state.player;
  ctx.fillStyle = '#dfeeff';
  ctx.beginPath();
  ctx.arc(x, y, radius, 0, Math.PI * 2);
  ctx.fill();

  ctx.fillStyle = '#284b69';
  ctx.fillRect(x - 7, y - 10, 14, 18);

  ctx.strokeStyle = '#8a5a2c';
  ctx.lineWidth = 5;
  ctx.beginPath();
  ctx.moveTo(x + 9, y + 8);
  ctx.lineTo(x + 20, y + 24);
  ctx.stroke();
}

function drawSeller() {
  const x = world.width - 120;
  const y = world.groundY - 40;
  ctx.fillStyle = '#5c4439';
  ctx.fillRect(x, y, 90, 42);
  ctx.fillStyle = '#d5b56a';
  ctx.fillRect(x + 8, y + 12, 74, 18);
  ctx.fillStyle = '#ffffff';
  ctx.font = 'bold 12px Arial';
  ctx.fillText('Vender', x + 24, y + 26);
}

function drawHud() {
  ctx.fillStyle = 'rgba(10, 18, 28, 0.35)';
  ctx.fillRect(20, 18, 175, 80);
  ctx.fillStyle = '#f8f9ff';
  ctx.font = 'bold 16px Arial';
  ctx.fillText('Madeira: ' + state.wood, 32, 44);
  ctx.fillText('Dinheiro: $' + state.money, 32, 66);
  ctx.fillText('Carga: ' + state.inventory + '/' + getCapacity(), 32, 88);

  if (state.hoverFlash > 0) {
    ctx.strokeStyle = 'rgba(255,255,255,0.7)';
    ctx.lineWidth = 2;
    ctx.beginPath();
    ctx.arc(state.player.x + 14, state.player.y - 6, 16 + (10 - state.hoverFlash), 0, Math.PI * 2);
    ctx.stroke();
  }
}

function updatePlayer() {
  let dx = 0;
  let dy = 0;
  if (controls['KeyW'] || controls['ArrowUp']) dy -= 1;
  if (controls['KeyS'] || controls['ArrowDown']) dy += 1;
  if (controls['KeyA'] || controls['ArrowLeft']) dx -= 1;
  if (controls['KeyD'] || controls['ArrowRight']) dx += 1;

  if (dx !== 0 || dy !== 0) {
    const len = Math.hypot(dx, dy) || 1;
    const speed = getCurrentAxe().speed;
    state.player.x += (dx / len) * speed;
    state.player.y += (dy / len) * speed;
  }

  state.player.x = clamp(state.player.x, 20, world.width - 20);
  state.player.y = clamp(state.player.y, 100, world.groundY - 20);
}

function updateTreeSimulation() {
  for (const tree of state.trees) {
    updateTreePhysics(tree);
  }

  if (state.saleCooldown > 0) state.saleCooldown -= 1;
  if (state.hoverFlash > 0) state.hoverFlash -= 1;
}

function sellWood() {
  if (state.wood <= 0 || state.saleCooldown > 0) return;
  const payout = state.wood * (8 + state.sawmillLevel * 4);
  state.money += payout;
  state.wood = 0;
  state.inventory = 0;
  state.saleCooldown = 20;
}

function updateHudText() {
  moneyValue.textContent = '$' + state.money;
  woodValue.textContent = String(state.wood);
  carValue.textContent = state.cartLevel === 0 ? 'Nenhum' : state.cartLevel === 1 ? 'Carro' : 'Carro Melhorado';
  currentAxeName.textContent = getCurrentAxe().name;
  speedValue.textContent = getCurrentAxe().speed.toFixed(1);
  capacityValue.textContent = String(getCapacity());
}

function renderShop() {
  axeShop.innerHTML = '';
  axes.forEach((axe, index) => {
    const item = document.createElement('div');
    item.className = 'shop-item' + (index === state.axeIndex ? ' active' : '');

    const row = document.createElement('div');
    row.className = 'item-row';
    row.innerHTML = `<strong>${axe.name}</strong><span>$${axe.cost}</span>`;

    const button = document.createElement('button');
    button.textContent = index === state.axeIndex ? 'Equipado' : 'Comprar';
    button.disabled = index === state.axeIndex || state.money < axe.cost;
    button.addEventListener('click', () => {
      if (state.money >= axe.cost) {
        state.money -= axe.cost;
        state.axeIndex = index;
        updateHudText();
        renderShop();
      }
    });

    item.appendChild(row);
    item.appendChild(button);
    axeShop.appendChild(item);
  });

  upgradeShop.innerHTML = '';
  const upgradesList = [
    { key: 'cart', info: upgrades.cart },
    { key: 'cart2', info: upgrades.cart2 },
    { key: 'sawmill', info: upgrades.sawmill },
  ];

  upgradesList.forEach((entry) => {
    const item = document.createElement('div');
    item.className = 'shop-item';

    const row = document.createElement('div');
    row.className = 'item-row';
    row.innerHTML = `<strong>${entry.info.name}</strong><span>$${entry.info.cost}</span>`;

    const button = document.createElement('button');
    button.textContent = 'Comprar';
    button.disabled = state.money < entry.info.cost;
    button.addEventListener('click', () => {
      if (state.money >= entry.info.cost) {
        state.money -= entry.info.cost;
        if (entry.key === 'cart') state.cartLevel = 1;
        if (entry.key === 'cart2') state.cartLevel = 2;
        if (entry.key === 'sawmill') state.sawmillLevel += 1;
        updateHudText();
        renderShop();
      }
    });

    item.appendChild(row);
    item.appendChild(button);
    upgradeShop.appendChild(item);
  });
}

function handleClick(event) {
  const rect = canvas.getBoundingClientRect();
  const scaleX = canvas.width / rect.width;
  const scaleY = canvas.height / rect.height;
  const x = (event.clientX - rect.left) * scaleX;
  const y = (event.clientY - rect.top) * scaleY;

  const success = chopNearestBranch(x, y);
  if (success) {
    updateHudText();
    renderShop();
  }
}

function drawBackgroundDecor() {
  for (let i = 0; i < 9; i++) {
    const x = 30 + i * 100;
    const y = 200 + (i % 2) * 20;
    ctx.fillStyle = '#6fb37d';
    ctx.beginPath();
    ctx.arc(x, y, 18, 0, Math.PI * 2);
    ctx.fill();
  }
}

function render() {
  ctx.clearRect(0, 0, world.width, world.height);
  drawBackgroundDecor();
  drawGround();
  renderTrees();
  drawSeller();
  drawPlayer();
  drawHud();
}

function gameLoop() {
  updatePlayer();
  updateTreeSimulation();
  render();
  requestAnimationFrame(gameLoop);
}

window.addEventListener('keydown', (event) => {
  controls[event.code] = true;
  if (event.code === 'KeyE') {
    sellWood();
    updateHudText();
  }
});

window.addEventListener('keyup', (event) => {
  controls[event.code] = false;
});

canvas.addEventListener('click', handleClick);

seedWorld();
renderShop();
updateHudText();
requestAnimationFrame(gameLoop);

// Seller zone bonus check
setInterval(() => {
  if (state.player.x > world.width - 150 && state.player.y > world.groundY - 90) {
    sellWood();
    updateHudText();
  }
}, 150);

// Quick start value for better feel
state.money = 60;
updateHudText();

console.log('Wood Rush carregado com sucesso.');
























































































































































































































































































































































































































