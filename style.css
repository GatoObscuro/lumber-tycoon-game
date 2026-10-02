:root {
  --bg: #09141d;
  --panel: rgba(15, 26, 39, 0.95);
  --panel-border: #1c3858;
  --gold: #f8d76c;
  --accent: #4ec5ff;
  --green: #57d27d;
  --text: #e8f3ff;
  --muted: #afcbe5;
  --danger: #ff6a6a;
}

* {
  box-sizing: border-box;
}

html, body {
  margin: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(180deg, #09141d 0%, #111d2f 100%);
  color: var(--text);
  font-family: Arial, Helvetica, sans-serif;
}

body {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 18px;
}

.game-shell {
  width: min(1500px, 100%);
  height: min(92vh, 760px);
  display: grid;
  grid-template-columns: 260px minmax(600px, 1fr) 260px;
  gap: 18px;
}

.panel {
  background: var(--panel);
  border: 2px solid var(--panel-border);
  border-radius: 16px;
  box-shadow: 0 20px 30px rgba(0,0,0,0.25);
  padding: 16px;
}

.left-panel, .right-panel {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

h1 {
  margin: 0;
  color: var(--gold);
  font-size: 2rem;
  letter-spacing: 1px;
}

.stat-box {
  display: grid;
  gap: 10px;
}

.stat {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: rgba(255,255,255,0.025);
  border: 1px solid rgba(255,255,255,0.08);
  border-radius: 12px;
  padding: 10px 12px;
}

.stat span {
  color: var(--muted);
  font-size: 0.9rem;
}

.stat strong {
  color: var(--gold);
}

.label {
  margin-top: 4px;
  color: #8fc7ff;
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 1.2px;
  text-transform: uppercase;
}

.shop-list {
  display: grid;
  gap: 10px;
}

.shop-item {
  background: rgba(255,255,255,0.025);
  border: 1px solid rgba(255,255,255,0.08);
  border-radius: 12px;
  padding: 10px 12px;
}

.shop-item.active {
  border-color: rgba(94, 214, 130, 0.8);
  box-shadow: inset 0 0 0 1px rgba(94, 214, 130, 0.4);
}

.item-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 8px;
}

.item-row strong {
  font-size: 0.96rem;
}

.item-row span {
  color: var(--gold);
  font-size: 0.9rem;
}

.shop-item button {
  width: 100%;
  border: none;
  border-radius: 8px;
  background: linear-gradient(180deg, #f1d36d 0%, #f3b63e 100%);
  color: #21160b;
  font-weight: 700;
  padding: 8px 10px;
  cursor: pointer;
}

.shop-item button:disabled {
  opacity: 0.55;
  cursor: not-allowed;
}

.game-area {
  background: rgba(13, 21, 31, 0.8);
  border: 2px solid var(--panel-border);
  border-radius: 18px;
  box-shadow: inset 0 0 18px rgba(255,255,255,0.04), 0 20px 40px rgba(0,0,0,0.25);
  display: flex;
  align-items: center;
  justify-content: center;
}

canvas {
  width: 100%;
  height: 100%;
  display: block;
  background: linear-gradient(180deg, #8bd8ff 0%, #bfe9ff 18%, #d8f4ff 40%, #63b96c 41%, #4e9455 100%);
  border-radius: 16px;
}

.right-panel p,
.right-panel li {
  color: var(--text);
  line-height: 1.5;
  margin: 0;
}

.right-panel ul {
  margin: 0;
  padding-left: 20px;
}

.status-list {
  display: grid;
  gap: 8px;
  color: var(--text);
}

@media (max-width: 1100px) {
  .game-shell {
    grid-template-columns: 1fr;
    height: auto;
  }

  .game-area {
    min-height: 520px;
  }
}
