# Rakha-
<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport"
content="width=device-width, initial-scale=1.0">

<title>Stick War Fixed</title>

<style>

*{
    margin:0;
    padding:0;
    box-sizing:border-box;
    font-family:Arial;
}

body{
    overflow:hidden;
    background:#0f172a;
}

canvas{
    display:block;
    background:#1e293b;
}

.ui{

    position:absolute;

    top:10px;

    left:10px;

    color:white;

    font-size:20px;

    line-height:32px;
}

.controls{

    position:absolute;

    bottom:20px;

    left:50%;

    transform:translateX(-50%);

    display:flex;

    gap:10px;
}

button{

    padding:14px 20px;

    border:none;

    border-radius:10px;

    background:#3b82f6;

    color:white;

    font-size:18px;

    font-weight:bold;
}

#restartBtn{

    position:absolute;

    top:60%;

    left:50%;

    transform:translateX(-50%);

    display:none;

    background:#22c55e;
}

</style>
</head>

<body>

<div class="ui">

Gold:
<span id="goldText">200</span><br>

Base HP:
<span id="baseText">300</span><br>

Enemy Base:
<span id="enemyBaseText">300</span>

</div>

<canvas id="game"></canvas>

<div class="controls">

<button id="spawnMiner">
Miner (50)
</button>

<button id="spawnSword">
Swordman (100)
</button>

</div>

<button id="restartBtn">
PLAY AGAIN
</button>

<script>

const canvas =
document.getElementById("game");

const ctx =
canvas.getContext("2d");

canvas.width = innerWidth;
canvas.height = innerHeight;

const goldText =
document.getElementById("goldText");

const baseText =
document.getElementById("baseText");

const enemyBaseText =
document.getElementById("enemyBaseText");

const restartBtn =
document.getElementById("restartBtn");


// ================= GAME =================

let gold = 200;

let baseHp = 300;

let enemyBaseHp = 300;

let gameOver = false;


// ================= BASE =================

const playerBase = {

    x:80,
    y:canvas.height/2
};

const enemyBase = {

    x:canvas.width - 80,
    y:canvas.height/2
};


// ================= GOLD MINE =================

const goldMine = {

    x:canvas.width/2,
    y:canvas.height/2
};


// ================= UNITS =================

const playerUnits = [];
const enemyUnits = [];


// ================= SPAWN PLAYER =================

document
.getElementById("spawnMiner")
.addEventListener(
"click",
()=>{

    if(gold >= 50){

        gold -= 50;

        playerUnits.push({

            type:"miner",

            x:140,
            y:canvas.height/2 + 50,

            hp:60,

            maxHp:60,

            speed:1.5,

            damage:0,

            attackRange:0,

            color:"#facc15",

            mining:false,

            returnGold:false
        });
    }
});

document
.getElementById("spawnSword")
.addEventListener(
"click",
()=>{

    if(gold >= 100){

        gold -= 100;

        playerUnits.push({

            type:"sword",

            x:140,
            y:canvas.height/2 - 50,

            hp:100,

            maxHp:100,

            speed:2,

            damage:1.5,

            attackRange:45,

            attackCooldown:0,

            color:"#3b82f6"
        });
    }
});


// ================= SPAWN ENEMY =================

function spawnEnemy(){

    enemyUnits.push({

        type:"enemy",

        x:canvas.width - 140,
        y:canvas.height/2,

        hp:100,

        maxHp:100,

        speed:1.8,

        damage:0.4,

        attackRange:45,

        attackCooldown:0,

        color:"#ef4444"
    });
}

setInterval(()=>{

    if(!gameOver){

        spawnEnemy();
    }

},4000);


// ================= DRAW =================

function drawCircle(x,y,r,color){

    ctx.fillStyle = color;

    ctx.beginPath();

    ctx.arc(x,y,r,0,Math.PI*2);

    ctx.fill();
}


// ================= DRAW BASE =================

function drawBases(){

    ctx.fillStyle = "#3b82f6";

    ctx.fillRect(
        playerBase.x - 40,
        playerBase.y - 100,
        80,
        200
    );

    ctx.fillStyle = "#ef4444";

    ctx.fillRect(
        enemyBase.x - 40,
        enemyBase.y - 100,
        80,
        200
    );

    ctx.fillStyle = "#facc15";

    ctx.fillRect(
        goldMine.x - 30,
        goldMine.y - 30,
        60,
        60
    );
}


// ================= PLAYER =================

function updatePlayerUnits(){

    for(let i=playerUnits.length-1;i>=0;i--){

        const unit = playerUnits[i];

        // ================= MINER =================

        if(unit.type === "miner"){

            if(!unit.mining && !unit.returnGold){

                unit.x += unit.speed;

                if(unit.x >= goldMine.x - 30){

                    unit.mining = true;
                }
            }

            else if(unit.mining){

                unit.mineTime =
                (unit.mineTime || 0) + 1;

                if(unit.mineTime > 120){

                    unit.mineTime = 0;

                    unit.mining = false;

                    unit.returnGold = true;
                }
            }

            else if(unit.returnGold){

                unit.x -= unit.speed;

                if(unit.x <= 140){

                    gold += 50;

                    unit.returnGold = false;
                }
            }
        }

        // ================= SWORDMAN =================

        if(unit.type === "sword"){

            let nearestEnemy = null;

            let nearestDist = 99999;

            for(let enemy of enemyUnits){

                const dist =
                Math.abs(enemy.x - unit.x);

                if(dist < nearestDist){

                    nearestDist = dist;

                    nearestEnemy = enemy;
                }
            }

            // cooldown

            if(unit.attackCooldown > 0){

                unit.attackCooldown--;
            }

            // lawan musuh

            if(nearestEnemy){

                const dist =
                Math.abs(
                    nearestEnemy.x - unit.x
                );

                if(dist > unit.attackRange){

                    unit.x += unit.speed;

                }else{

                    // ATTACK

                    if(unit.attackCooldown <= 0){

                        nearestEnemy.hp -=
                        unit.damage;

                        unit.attackCooldown = 25;
                    }
                }

            }else{

                // attack base

                if(unit.x <
                enemyBase.x - 60){

                    unit.x += unit.speed;

                }else{

                    enemyBaseHp -= 0.2;
                }
            }
        }

        // DRAW

        drawCircle(
            unit.x,
            unit.y,
            20,
            unit.color
        );

        // HP BAR

        ctx.fillStyle = "red";

        ctx.fillRect(
            unit.x - 20,
            unit.y - 35,
            40,
            5
        );

        ctx.fillStyle = "lime";

        ctx.fillRect(
            unit.x - 20,
            unit.y - 35,
            (unit.hp/unit.maxHp)*40,
            5
        );

        // DEAD

        if(unit.hp <= 0){

            playerUnits.splice(i,1);
        }
    }
}


// ================= ENEMY =================

function updateEnemyUnits(){

    for(let i=enemyUnits.length-1;i>=0;i--){

        const unit = enemyUnits[i];

        let nearestPlayer = null;

        let nearestDist = 99999;

        for(let p of playerUnits){

            const dist =
            Math.abs(p.x - unit.x);

            if(dist < nearestDist){

                nearestDist = dist;

                nearestPlayer = p;
            }
        }

        // cooldown

        if(unit.attackCooldown > 0){

            unit.attackCooldown--;
        }

        if(nearestPlayer){

            const dist =
            Math.abs(
                nearestPlayer.x - unit.x
            );

            if(dist > unit.attackRange){

                unit.x -= unit.speed;

            }else{

                if(unit.attackCooldown <= 0){

                    nearestPlayer.hp -=
                    unit.damage;

                    unit.attackCooldown = 25;
                }
            }

        }else{

            if(unit.x >
            playerBase.x + 60){

                unit.x -= unit.speed;

            }else{

                baseHp -= 0.2;
            }
        }

        // DRAW

        drawCircle(
            unit.x,
            unit.y,
            20,
            unit.color
        );

        // HP BAR

        ctx.fillStyle = "red";

        ctx.fillRect(
            unit.x - 20,
            unit.y - 35,
            40,
            5
        );

        ctx.fillStyle = "lime";

        ctx.fillRect(
            unit.x - 20,
            unit.y - 35,
            (unit.hp/unit.maxHp)*40,
            5
        );

        // DEAD

        if(unit.hp <= 0){

            enemyUnits.splice(i,1);

            gold += 25;
        }
    }
}


// ================= UI =================

function updateUI(){

    goldText.innerText =
    Math.floor(gold);

    baseText.innerText =
    Math.floor(baseHp);

    enemyBaseText.innerText =
    Math.floor(enemyBaseHp);
}


// ================= MESSAGE =================

function showMessage(text){

    ctx.fillStyle =
    "rgba(0,0,0,0.7)";

    ctx.fillRect(
        0,
        0,
        canvas.width,
        canvas.height
    );

    ctx.fillStyle = "white";

    ctx.font = "60px Arial";

    ctx.textAlign = "center";

    ctx.fillText(
        text,
        canvas.width/2,
        canvas.height/2
    );

    restartBtn.style.display =
    "block";
}


// ================= RESET =================

function resetGame(){

    gold = 200;

    baseHp = 300;

    enemyBaseHp = 300;

    playerUnits.length = 0;

    enemyUnits.length = 0;

    gameOver = false;

    restartBtn.style.display =
    "none";
}

restartBtn.addEventListener(
"click",
resetGame
);


// ================= LOOP =================

function gameLoop(){

    ctx.clearRect(
        0,
        0,
        canvas.width,
        canvas.height
    );

    drawBases();

    if(!gameOver){

        updatePlayerUnits();

        updateEnemyUnits();
    }

    updateUI();

    if(baseHp <= 0 && !gameOver){

        gameOver = true;

        showMessage("DEFEAT");
    }

    if(enemyBaseHp <= 0 && !gameOver){

        gameOver = true;

        showMessage("VICTORY");
    }

    requestAnimationFrame(gameLoop);
}

gameLoop();

</script>

</body>
</html>
