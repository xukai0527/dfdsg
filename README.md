# dfdsg
const app = document.getElementById('app');

// 游戏状态
let state = {
    diskCount: 3,
    mode: 'manual', // manual or auto
    towers: [[], [], []], // 存储圆盘数据的逻辑数组
    isDragging: false,
    draggedDiskData: null, // 当前拖拽的圆盘对象 {element, size, fromTowerIndex}
    moveCount: 0,
    timer: null,
    secondsElapsed: 0,
    isPaused: false
};

const colors = ['#e74c3c', '#3498db', '#2ecc71', '#f1c40f', '#9b59b6', '#e67e22', '#1abc9c', '#ecf0f1'];


function renderHome() {
    app.innerHTML = '';
    const homeDiv = document.createElement('div');
    homeDiv.className = 'home-screen';
    homeDiv.innerHTML = `
        <h1>汉诺塔</h1>
        <div class="setting-item">
            <label>圆盘数量:</label>
            <input type="number" id="diskInput" value="3" min="3" max="8">
        </div>
        <div class="setting-item">
            <label>游戏模式:</label>
            <select id="modeSelect">
                <option value="manual">手动游玩</option>
                <option value="auto">自动演示</option>
            </select>
        </div>
        <button class="btn btn-primary" id="startBtn">开始游戏</button>
    `;
    app.appendChild(homeDiv);

    document.getElementById('startBtn').addEventListener('click', () => {
        const count = parseInt(document.getElementById('diskInput').value);
        const mode = document.getElementById('modeSelect').value;
        if (count < 3 || count > 8) {
            alert("圆盘数量需在 3-8 之间");
            return;
        }
        startGame(count, mode);
    });
}

function renderGame() {
    app.innerHTML = '';
    const gameContainer = document.createElement('div');
    gameContainer.className = 'game-container';
    const header = document.createElement('div');
    header.className = 'game-header';
    header.innerHTML = `
        <span>步数: <span id="moveCounter">0</span></span>
        <span>时间: <span id="timer">00:00</span></span>
        <button class="btn btn-danger" id="pauseBtn">暂停</button>
    `;
    const gameArea = document.createElement('div');
    gameArea.className = 'game-area';
    gameArea.id = 'gameArea';
    for (let i = 0; i < 3; i++) {
        const tower = document.createElement('div');
        tower.className = 'tower';
        tower.dataset.index = i;
        tower.innerHTML = '<div class="peg"></div><div class="drop-indicator">✔</div>'; 
        gameArea.appendChild(tower);
    }

    gameContainer.appendChild(header);
    gameContainer.appendChild(gameArea);
    app.appendChild(gameContainer);

    document.getElementById('pauseBtn').addEventListener('click', togglePause);
}

function renderWinModal() {
    const overlay = document.createElement('div');
    overlay.className = 'modal-overlay';
    overlay.innerHTML = `
        <div class="modal">
            <h2>恭喜通关! 🎉</h2>
            <p>总步数: ${state.moveCount}</p >
            <p>用时: ${formatTime(state.secondsElapsed)}</p >
            <button class="btn btn-primary" onclick="location.reload()">返回首页</button>
        </div>
    `;
    app.appendChild(overlay);

    // 简单的礼花特效
    for(let i=0; i<50; i++) {
        const confetti = document.createElement('div');
        confetti.className = 'confetti';
        confetti.style.left = Math.random() * 100 + 'vw';
        confetti.style.backgroundColor = colors[Math.floor(Math.random() * colors.length)];
        confetti.style.animationDuration = (Math.random() * 3 + 2) + 's';
        overlay.appendChild(confetti);
    }
}


function startGame(count, mode) {
    state.diskCount = count;
    state.mode = mode;
    state.towers = [[], [], []];
    state.moveCount = 0;
    state.secondsElapsed = 0;
    state.isPaused = false;
    for (let i = count; i >= 1; i--) {
        state.towers[0].push({ size: i, color: colors[i % colors.length] });
    }

    renderGame();
    updateBoard(); 
    startTimer();

    if (mode === 'auto') {

        setTimeout(() => {
            solveAuto(count, 0, 2, 1);
        }, 1000);
    }
}

function updateBoard() {
    const gameArea = document.getElementById('gameArea');
    
    const towersEls = gameArea.querySelectorAll('.tower');
    towersEls.forEach(tower => {
        const disks = tower.querySelectorAll('.disk');
        disks.forEach(d => d.remove());
    });

   
    state.towers.forEach((towerData, towerIndex) => {
        const towerEl = towersEls[towerIndex];
        towerData.forEach((diskData, diskIndex) => {
            const diskEl = document.createElement('div');
            diskEl.className = 'disk';
            diskEl.style.width = (diskData.size * 30) + 'px';
            diskEl.style.backgroundColor = diskData.color;
         
            positionDisk(diskEl, towerIndex, diskIndex);

            if (state.mode === 'manual') {
                addDragEvents(diskEl, diskData, towerIndex);
            }
            towerEl.appendChild(diskEl);
        });
    });
}


function getElementPosition(towerIndex, diskIndex) {
    const towerEl = document.querySelectorAll('.tower')[towerIndex];
    const towerRect = towerEl.getBoundingClientRect();
    const gameAreaRect = document.getElementById('gameArea').getBoundingClientRect();

  
    const centerX = towerRect.left + towerRect.width / 2 - gameAreaRect.left;

    const y = towerRect.height - 10 - (diskIndex + 1) * 30;

    return {
        left: centerX - (towerRect.width/2), 
        x: centerX - (diskData.size * 15), 
        y: y
    };
}


function positionDisk(diskEl, towerIndex, diskIndex) {
 
    const diskData = state.towers[towerIndex][diskIndex];
    const towerEl = document.querySelectorAll('.tower')[towerIndex];
    const gameArea = document.getElementById('gameArea');
    const gameRect = gameArea.getBoundingClientRect();
    const towerRect = towerEl.getBoundingClientRect();
    const centerOfTower = towerRect.left + (towerRect.width / 2) - gameRect.left;
    const diskWidth = diskData.size * 30;
    const left = centerOfTower - (diskWidth / 2);

    const top = towerRect.height - 10 - (diskIndex + 1) * 30;

    diskEl.style.left = left + 'px';
    diskEl.style.top = top + 'px';
}

function refreshPositions() {
    const towersEls = document.querySelectorAll('.tower');
    state.towers.forEach((towerData, tIdx) => {
        towerData.forEach((diskData, dIdx) => {

        });
    });
}

function addDragEvents(diskEl, diskData, towerIndex) {
    let startX, startY, initialLeft, initialTop;

    const onMouseDown = (e) => {
        if (state.isPaused) return;
        e.preventDefault(); 
        state.isDragging = true;
        state.draggedDiskData = { element: diskEl, size: diskData.size, fromTowerIndex: towerIndex };
        const gameArea = document.getElementById('gameArea');
        gameArea.appendChild(diskEl);
        const rect = diskEl.getBoundingClientRect();
        const gameRect = gameArea.getBoundingClientRect();

        startX = e.clientX;
        startY = e.clientY;
        initialLeft = rect.left - gameRect.left;
        initialTop = rect.top - gameRect.top;

        diskEl.classList.add('dragging');
        diskEl.style.zIndex = 1000;
    document.addEventListener('mousemove', onMouseMove);
        document.addEventListener('mouseup', onMouseUp);
    };

    const onMouseMove = (e) => {
        if (!state.isDragging) return;

        const gameArea = document.getElementById('gameArea');
        const gameRect = gameArea.getBoundingClientRect();

        const currentLeft = e.clientX - gameRect.left - (startX - gameRect.left - initialLeft);
        const currentTop = e.clientY - gameRect.top - (startY - gameRect.top - initialTop);

        diskEl.style.left = currentLeft + 'px';
        diskEl.style.top = currentTop + 'px';
        checkDropTarget(e.clientX, e.clientY);
    };

    const onMouseUp = (e) => {
        if (!state.isDragging) return;
        state.isDragging = false;        document.removeEventListener('mousemove', onMouseMove);
        document.removeEventListener('mouseup', onMouseUp);       document.querySelectorAll('.tower').forEach(t => {
            t.classList.remove('can-drop', 'cannot-drop');
        });

        // 查找释放的目标塔
        const targetTowerEl = document.elementFromPoint(e.clientX, e.clientY)?.closest('.tower');

        if (targetTowerEl) {
            const targetIndex = parseInt(targetTowerEl.dataset.index);
            // 尝试移动
            attemptMove(state.draggedDiskData.fromTowerIndex, targetIndex);
        } else {
            // 放回原处，如果不在塔上
            // 注意：attemptMove 会处理 UI 更新，如果失败，我们需要手动刷新位置
            // 但 attemptMove 里如果返回 false，我们得把 DOM 放回去
            // 为了简单，我们在 attemptMove 成功才移除事件，失败则依靠 updateBoard 修正
        }

        diskEl.classList.remove('dragging');
        diskEl.style.zIndex = ''; 
        state.draggedDiskData = null;  
        updateBoard();
    };

    diskEl.addEventListener('mousedown', onMouseDown);
}

function checkDropTarget(mouseX, mouseY) {
 document.querySelectorAll('.tower').forEach(t => {
        t.classList.remove('can-drop', 'cannot-drop');
    });

    if (!state.draggedDiskData) return;

    const targetTowerEl = document.elementFromPoint(mouseX, mouseY)?.closest('.tower');
    if (targetTowerEl) {
        const targetIndex = parseInt(targetTowerEl.dataset.index);
        const fromIdx = state.draggedDiskData.fromTowerIndex;
        if (targetIndex === fromIdx) return;      
        const targetTower = state.towers[targetIndex];
        const draggedSize = state.draggedDiskData.size;

        if (targetTower.length === 0 || targetTower[targetTower.length - 1].size > draggedSize) {
            targetTowerEl.classList.add('can-drop');
        } else {
            targetTowerEl.classList.add('cannot-drop');
        }
    }
}
// 尝试移动：返回 Promise，成功 resolve，失败 reject
function attemptMove(fromIdx, toIdx) {
    return new Promise((resolve, reject) => {
        if (fromIdx === toIdx) { reject('Same Tower'); return; }
        if (state.towers[fromIdx].length === 0) { reject('Empty Source'); return; }

        const disk = state.towers[fromIdx][state.towers[fromIdx].length - 1];
        const targetTower = state.towers[toIdx];
        if (targetTower.length > 0 && targetTower[targetTower.length - 1].size < disk.size) {
            reject('Invalid Move');
            return;
        }     
        animateMove(fromIdx, toIdx).then(() => {         
            state.towers[fromIdx].pop();
            targetTower.push(disk);
            state.moveCount++;
            document.getElementById('moveCounter').innerText = state.moveCount;
            resolve();
            checkWin();
        });
    });
}
function animateMove(fromIdx, toIdx) {
    return new Promise(resolve => {
        let diskEl;
        if (state.mode === 'manual' && state.isDragging) {
            diskEl = state.draggedDiskData.element;
        } else {              
            const towers = document.querySelectorAll('.tower');
            const fromTowerEl = towers[fromIdx];
            const disks = fromTowerEl.querySelectorAll('.disk');
            diskEl = disks[disks.length - 1];
        }

        if (!diskEl) { resolve(); return; }
        const gameArea = document.getElementById('gameArea');
        if (diskEl.parentElement !== gameArea) {
            gameArea.appendChild(diskEl);
        }
        const startRect = diskEl.getBoundingClientRect();
        const gameRect = gameArea.getBoundingClientRect();
        const startX = startRect.left - gameRect.left;
        const startY = startRect.top - gameRect.top;
        const targetDiskIndex = state.towers[toIdx].length;

        const targetTowerEl = document.querySelectorAll('.tower')[toIdx];
        const towerRect = targetTowerEl.getBoundingClientRect();
        const centerOfTower = towerRect.left + (towerRect.width / 2) - gameRect.left;
        const diskWidth = parseInt(diskEl.style.width);
        const endX = centerOfTower - (diskWidth / 2);
        const endY = towerRect.height - 10 - (targetDiskIndex + 1) * 30;
        diskEl.style.transition = 'top 0.5s ease-in-out, left 0.5s ease-in-out';
        diskEl.style.left = endX + 'px';
        diskEl.style.top = endY + 'px';
        const onTransitionEnd = () => {
            diskEl.style.transition = ''; 
            diskEl.removeEventListener('transitionend', onTransitionEnd);

            if (state.mode === 'auto') {
                updateBoard();
            }
            resolve();
        };
        diskEl.addEventListener('transitionend', onTransitionEnd);
    });
}
async function solveAuto(n, from, to, aux) {
    if (state.isPaused) return; // 暂停检查
    if (n === 0) return;

    await sleep(600); // 演示间隔
    if (state.isPaused) return;

    await solveAuto(n - 1, from, aux, to);
    if (state.isPaused) return;

    await attemptMove(from, to);
    if (state.isPaused) return;

    await solveAuto(n - 1, aux, to, from);
}

function sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}
function startTimer() {
    clearInterval(state.timer);
    state.timer = setInterval(() => {
        if (!state.isPaused) {
            state.secondsElapsed++;
            document.getElementById('timer').innerText = formatTime(state.secondsElapsed);
        }
    }, 1000);
}

function formatTime(s) {
    const min = Math.floor(s / 60).toString().padStart(2, '0');
    const sec = (s % 60).toString().padStart(2, '0');
    return `${min}:${sec}`;
}

function togglePause() {
    state.isPaused = !state.isPaused;
    const btn = document.getElementById('pauseBtn');
    btn.innerText = state.isPaused ? '继续' : '暂停';

    if (!state.isPaused) {
        if (state.mode === 'auto') {
        }
    }
}

function checkWin() {
    if (state.towers[2].length === state.diskCount) {
        clearInterval(state.timer);
        setTimeout(renderWinModal, 500);
    }
}
renderHome();
