// ==UserScript==
// @name       🎂Cheese Cake Mod🎂
// @namespace    -
// @version    1
// @description Proper Mod For Baking :/
// @author       CheeseCake's Friend
// @match        *://sandbox.moomoo.io/*
// @match        *://moomoo.io/*
// @grant        none
// @require https://greasyfork.org/scripts/368273-msgpack/code/msgpack.js?version=598723
// @require http://code.jquery.com/jquery-3.3.1.min.js
// @require https://code.jquery.com/ui/1.12.0/jquery-ui.min.js
// @require https://cdnjs.cloudflare.com/ajax/libs/jquery-confirm/3.3.0/jquery-confirm.min.js
// @require https://cdn.jsdelivr.net/gh/emn178/js-sha3/build/sha3.min.js

// ==/UserScript==


let mouseX;
let mouseY;

let width;
let height;

setInterval(() => {
   if(clanToggle == 1) {
        doNewSend(["9", [null]]);
        doNewSend(["8", [animate(false, 5)]])
    }
    doNewSend(["testing", [6]]);
}, 200);

setInterval(() => {
    if(messageToggle == 1) {
        doNewSend(["ch", [animate(true, 5)]])
    }
}, 200);

setInterval(() => {
    if(autoaim == true) {
        doNewSend(["2", [nearestEnemyAngle]]);
    }
}, 0);

setInterval(() => {
    if(autoprimary == true) {
        doNewSend(["5", [primary, true]]);
    }
}, 0);

setInterval(() => {
    if(autosecondary == true) {
        doNewSend(["5", [secondary, true]]);
    }
}, 0);

setInterval(() => {
    if(hatToggle == 1) {
        if(oldHat != normalHat) {
            hat(normalHat);
            console.log("Tried. - Hat")
        }
        if(oldAcc != normalAcc) {
            acc(normalAcc);
            console.log("Tried. - Acc")
        }
        oldHat = normalHat;
        oldAcc = normalAcc
    }
}, 25);

function normal() {
    hat(normalHat);
    acc(normalAcc);
}

function aim(x, y){
     var cvs = document.getElementById("gameCanvas");
     cvs.dispatchEvent(new MouseEvent("mousemove", {
         clientX: x,
         clientY: y

     }));
}

let coreURL = new URL(window.location.href);
window.sessionStorage.force = coreURL.searchParams.get("fc");

var AIH = true;
var nearestEnemy;
var nearestEnemyAngle;
var isEnemyNear;
var primary;
var secondary;
var foodType;
var wallType;
var spikeType;
var millType;
var mineType;
var boostType;
var turretType;
var spawnpadType;
var autoaim = false;
var autoprimary = false;
var autosecondary = false;
var tick = 1;
var oldHat;
var oldAcc;
var enemiesNear;
var normalHat;
var normalAcc;
var ws;
var msgpack5 = msgpack;
var boostDir;
let myPlayer = {
    id: null,
    x: null,
    y: null,
    dir: null,
    object: null,
    weapon: null,
    clan: null,
    isLeader: null,
    hat: null,
    accessory: null,
    isSkull: null
};

let healSpeed = 60;
var messageToggle = 0;
var clanToggle = 0;
let healToggle = 1;
let hatToggle = 1;



document.msgpack = msgpack;
function n(){
     this.buffer = new Uint8Array([0]);
     this.buffer.__proto__ = new Uint8Array;
     this.type = 0;
}

const CanvasAPI = document.getElementById("gameCanvas")
CanvasAPI.addEventListener("mousedown", buttonPressD, false);

function buttonPressD(e) {
    if (e.button == 2) {
            doNewSend(["13c", [1, 40, 0]]);
            doNewSend(["13c", [0, 40, 0]]);
            doNewSend(["13c", [0, 0, 1]]);
            doNewSend(["13c", [1, 21, 1]]);
            doNewSend(["13c", [0, 21, 1]]);
            doNewSend(["7", [1]]);
        setTimeout( () => {
            doNewSend(["13c", [0, 0, 0]]);
            doNewSend(["7", [1]]);
            doNewSend(["13c", [0, 11, 1]]);
            if (myPlayer.y < 2400){
                doNewSend(["13c", [0, 15, 0]]);
            } else if (myPlayer.y > 6850 && myPlayer.y < 7550){
                doNewSend(["13c", [0, 31, 0]]);
            } else {
	            doNewSend(["13c", [0, 12, 0]]);
            }
        }, 120);
    }
    if (e.button == 0) {
            doNewSend(["13c", [1, 7, 0]]);
            doNewSend(["13c", [0, 7, 0]]);
            doNewSend(["13c", [0, 0, 1]]);
            doNewSend(["13c", [1, 18, 1]]);
            doNewSend(["13c", [0, 18, 1]]);
            doNewSend(["7", [1]]);
        setTimeout( () => {
            doNewSend(["13c", [0, 0, 0]]);
            doNewSend(["13c", [0, 11, 1]]);
            if (myPlayer.y < 2400){
                doNewSend(["13c", [0, 15, 0]]);
            } else if (myPlayer.y > 6850 && myPlayer.y < 7550){
                doNewSend(["13c", [0, 31, 0]]);
            } else {
	            doNewSend(["13c", [0, 12, 0]]);
            }
            doNewSend(["7", [1]]);
        }, 120);
    }
}

WebSocket.prototype.oldSend = WebSocket.prototype.send;
WebSocket.prototype.send = function(m){
    if (!ws){
        document.ws = this;

        ws = this;
        socketFound(this);
    }
    this.oldSend(m);
};


function socketFound(socket){
    socket.addEventListener('message', function(message){
        handleMessage(message);
    });
}

function handleMessage(m){
    let temp = msgpack5.decode(new Uint8Array(m.data));
    let data;
    if(temp.length > 1) {
        data = [temp[0], ...temp[1]];
        if (data[1] instanceof Array){
            data = data;
        }
    } else {
      data = temp;
    }
    let item = data[0];
    if(!data) {return};

    if(item === "io-init") {
            let cvs = document.getElementById("gameCanvas");
            width = cvs.clientWidth;
            height = cvs.clientHeight;
            $(window).resize(function() {
                width = cvs.clientWidth;
                height = cvs.clientHeight;
            });
            cvs.addEventListener("mousemove", e => {
                mouseX = e.clientX;
                mouseY = e.clientY;
            });
        }

    if (item == "1" && myPlayer.id == null){
        myPlayer.id = data[1];
    }

    if (item == "33") {
        enemiesNear = [];
        for(let i = 0; i < data[1].length / 13; i++) {
            let playerInfo = data[1].slice(13*i, 13*i+13);
            if(playerInfo[0] == myPlayer.id) {
                myPlayer.x = playerInfo[1];
                myPlayer.y = playerInfo[2];
                myPlayer.dir = playerInfo[3];
                myPlayer.object = playerInfo[4];
                myPlayer.weapon = playerInfo[5];
                myPlayer.clan = playerInfo[7];
                myPlayer.isLeader = playerInfo[8];
                myPlayer.hat = playerInfo[9];
                myPlayer.accessory = playerInfo[10];
                myPlayer.isSkull = playerInfo[11];
            } else if(playerInfo[7] != myPlayer.clan || playerInfo[7] === null) {
                enemiesNear.push(playerInfo);
            }
        }
    }

    isEnemyNear = false;
    if(enemiesNear) {
        nearestEnemy = enemiesNear.sort((a,b) => dist(a, myPlayer) - dist(b, myPlayer))[0];
    }

    if(nearestEnemy) {
        nearestEnemyAngle = Math.atan2(nearestEnemy[2]-myPlayer.y, nearestEnemy[1]-myPlayer.x);
        if(Math.sqrt(Math.pow((myPlayer.y-nearestEnemy[2]), 2) + Math.pow((myPlayer.x-nearestEnemy[1]), 2)) < 0) {
            isEnemyNear = true;
            if(autoaim == false && myPlayer.hat != 7 && myPlayer.hat != 53) {
                normalHat = 6;
                if(primary != 8) {
                    normalAcc = 19
                }
            };
        }
    }
    if(isEnemyNear == false && autoaim == false) {
        normalAcc = 11;
        if (myPlayer.y < 2400){
            normalHat = 15;
        } else if (myPlayer.y > 6850 && myPlayer.y < 7550){
            normalHat = 31;
        } else {
	        normalHat = 12;
        }
    }
    if (!nearestEnemy) {
        nearestEnemyAngle = myPlayer.dir;
    }

    if (item == "h" && data[1] == myPlayer.id) {
        if (data[2] < 49 && data[2] > 0 && AIH == true) {
            doNewSend(["ch" ["Cheese Cake Antiinsta.."]]);
            place(foodType);
            place(foodType);
        }
    }
   if(item == "h" && data[1] == myPlayer.id) {
       if(data[2] < 100 && data[2] > 0 && healToggle == 1) {
           setTimeout( () => {
               place(foodType, null);
               place(foodType, null);
           }, healSpeed);

       }
   }
   update();
}


function doNewSend(sender){
    ws.send(new Uint8Array(Array.from(msgpack5.encode(sender))));
}

function acc(id) {
    doNewSend(["13c", [0, 0, 1]]);
    doNewSend(["13c", [0, id, 1]]);
}

function hat(id) {
    doNewSend(["13c", [0, id, 0]]);
}


function place(id, angle = Math.atan2(mouseY - height / 2, mouseX - width / 2)) {
    doNewSend(["5", [id, null]]);
    doNewSend(["c", [1, angle]]);
    doNewSend(["c", [0, angle]]);
    doNewSend(["5", [myPlayer.weapon, true]]);
}

function placeQ(id, angle = Math.atan2(mouseY - height / 2, mouseX - width / 2)) {
    doNewSend(["5", [id, null]]);
    doNewSend(["c", [1, boostDir]]);
    doNewSend(["c", [0, boostDir]]);
    doNewSend(["5", [myPlayer.weapon, true]]);
    doNewSend(["2", [nearestEnemyAngle]]);
}

function boostSpike() {
    if(boostDir == null) {
        boostDir = nearestEnemyAngle;
    }
    place(spikeType, boostDir - toRad(90));
    place(spikeType, boostDir + toRad(90));
    place(boostType, boostDir);
    doNewSend(["33", [boostDir]]);
}

var repeater = function(key, action, interval) {
    let _isKeyDown = false;
    let _intervalId = undefined;

    return {
        start(keycode) {
            if(keycode == key && document.activeElement.id.toLowerCase() !== 'chatbox') {
                _isKeyDown = true;
                if(_intervalId === undefined) {
                    _intervalId = setInterval(() => {
                        action();
                        if(!_isKeyDown){
                            clearInterval(_intervalId);
                            _intervalId = undefined;
                            console.log("claered");
                        }
                    }, interval);
                }
            }
        },

        stop(keycode) {
            if(keycode == key && document.activeElement.id.toLowerCase() !== 'chatbox') {
                _isKeyDown = false;
            }
        }
    };


}


const healer1 = repeater(51, () => {placeQ(foodType, boostDir);
                                    placeQ(foodType, boostDir);
                                    placeQ(foodType, boostDir)}, 50);
const healer2 = repeater(81, () => {placeQ(foodType, boostDir);
                                    placeQ(foodType, boostDir);
                                    placeQ(foodType, boostDir)}, 50);
const boostPlacer = repeater(70, () => {place(boostType)}, 0);
const spikePlacer = repeater(86, () => {place(spikeType)}, 0);
const millPlacer = repeater(78, () => {
place(millType, Math.atan2(mouseY - height / 2, mouseX - width / 2) + toRad(90));
place(millType, Math.atan2(mouseY - height / 2, mouseX - width / 2) + toRad(280));
place(millType, Math.atan2(mouseY - height / 2, mouseX - width / 2) + toRad(0));
place(millType)}, 0);
const turretPlacer = repeater(72, () => {place(turretType)}, 0);
const boostSpiker = repeater(71, () => {boostSpike}, 0);

document.addEventListener('keydown', (e)=>{
    spikePlacer.start(e.keyCode);
    healer1.start(e.keyCode);
    healer2.start(e.keyCode);
    boostPlacer.start(e.keyCode);
    boostSpiker.start(e.keyCode);
    millPlacer.start(e.keyCode);
    turretPlacer.start(e.keyCode);

    if (e.keyCode == 67 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        doNewSend(["13c", [1, 6, 0]]);
        doNewSend(["13c", [0, 6, 0]]);
        doNewSend(["13c", [0, 0, 1]]);
        doNewSend(["13c", [1, 19, 1]]);
        doNewSend(["13c", [0, 19, 1]]);
    }

    if (e.keyCode == 49 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        autosecondary = false;
        autoprimary = true;
        setTimeout( () => {
            autoprimary = false;
        }, 330);
    }

    if (e.keyCode == 50 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        autoprimary = false;
        autosecondary = true;
        setTimeout( () => {
            autosecondary = false;
        }, 330);
    }
    if (e.keyCode == 76 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        doNewSend(["ch", ['Cheese + Milk + Flower = ?']]);
        autoaim = true;
        doNewSend(["5", [secondary, true]]);
                         hat(30);
        doNewSend(["ch", [spam(true, 5)]])
        doNewSend(["13c", [0, 32, 0]]);
        doNewSend(["13c", [0, 21, 1]]);
        doNewSend(["13c", [0, 53, 0]]);
        doNewSend(["c", [1]]);

        setTimeout( () => {
            doNewSend(["13c", [0, 32, 19]]);
            doNewSend(["13c", [0, 21, 1]]);
            doNewSend(["13c", [0, 32, 0]]);
            doNewSend(["6", [12]]);

        }, 110);

        setTimeout( () => {
            doNewSend(["6", [15]]);

        }, 220);

        setTimeout( () => {
            doNewSend(["c", [0]]);
                       hat(57);
            doNewSend(["5", [primary, true]]);
            autoaim = false;
        }, 330);
    }
        if (e.keyCode == 32 && document.activeElement.id.toLowerCase() !== 'chatbox') {
            place(spikeType, myPlayer.dir + toRad(45));
            place(spikeType, myPlayer.dir - toRad(45));
        }
    if (e.keyCode == 82 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        doNewSend(["ch", ["You Like Cheese Cake?!"]]);
    }
    if(e.keyCode == 82 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        setTimeout( () => {
        doNewSend(["ch", ["....Adding Milk...."]]);
        doNewSend(["5", [secondary, true]]);
        }, 700)
    }
    if(e.keyCode == 82 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        setTimeout( () => {
        doNewSend(["ch", ["...Adding Cheese+Flower..."]]);
        doNewSend(["5", [secondary, true]]);
        }, 1300)
   }
    if(e.keyCode == 82 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        setTimeout( () => {
        doNewSend(["ch", ["      Cheese Cake Ready!     "]]);
        doNewSend(["5", [primary, true]]);
        }, 2500)
    }
    if (e.keyCode == 77 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        doNewSend(["13c", [1, 7, 0]]);
        doNewSend(["13c", [0, 7, 0]]);
        doNewSend(["13c", [0, 0, 1]]);
        doNewSend(["13c", [0, 19, 1]]);
    }

    if (e.keyCode == 90 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        doNewSend(["13c", [1, 40, 0]]);
        doNewSend(["13c", [0, 40, 0]]);
        doNewSend(["13c", [0, 0, 1]]);
        doNewSend(["13c", [0, 19, 1]]);
    }

    if (e.keyCode == 84 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        doNewSend(["13c", [1, 22, 0]]);
        doNewSend(["13c", [0, 22, 0]]);
        doNewSend(["13c", [0, 0, 1]]);
        doNewSend(["13c", [0, 19, 1]]);
    }

    if (e.keyCode == 9 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        doNewSend(["13c", [1, 11, 1]]);
        doNewSend(["13c", [0, 11, 1]]);
        doNewSend(["13c", [1, 12, 0]]);
        doNewSend(["13c", [0, 12, 0]]);
        doNewSend(["13c", [1, 31, 0]]);
        doNewSend(["13c", [1, 15, 0]]);
    }

    if(e.keyCode == 106 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        healToggle = (healToggle + 1) % 2;
    }

    if(e.keyCode == 40 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        doNewSend(["6", [5]]);
        doNewSend(["6", [17]]);
        doNewSend(["6", [31]]);
        doNewSend(["6", [23]]);
        doNewSend(["6", [9]]);
        doNewSend(["6", [38]]);
        doNewSend(["6", [28]]);
        doNewSend(["6", [13]]);
    }
    if (e.keyCode == 32 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        for (let e = 0; e < 3; e++) {
        place(spikeType, e);
        }
    }
    if (e.keyCode == 80 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        for (let i=0;i<4;i++){
             let angle = myPlayer.dir + toRad(i * 90);
             place(wallType, angle)
        }
    }
    if (e.keyCode == 73 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        for (let i=0;i<4;i++){
             let angle = myPlayer.dir + toRad(i * 90);
             place(boostType, angle)
        }
    }
    if (e.keyCode == 186 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        for (let i=0;i<4;i++){
             let angle = myPlayer.dir + toRad(i * 90);
             place(spikeType, angle)
        }
    }
    if (e.keyCode == 72 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        place(turretType, myPlayer.dir + toRad(45));
        place(turretType, myPlayer.dir - toRad(45));
    }

    if(e.keyCode == 188 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        doNewSend(["6", [7]]);
        doNewSend(["6", [17]]);
        doNewSend(["6", [31]]);
        doNewSend(["6", [27]]);
        doNewSend(["6", [10]]);
        doNewSend(["6", [38]]);
        doNewSend(["6", [4]]);
        doNewSend(["6", [15]]);
    }
    if(e.keyCode == 220 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        doNewSend(["6", [9]]);
        doNewSend(["6", [17]]);
        doNewSend(["6", [31]]);
        doNewSend(["6", [27]]);
        doNewSend(["6", [10]]);
        doNewSend(["6", [38]]);
        doNewSend(["6", [4]]);
        doNewSend(["6", [25]]);
    }
    if(e.keyCode == 190 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        doNewSend(["6", [7]]);
        doNewSend(["6", [17]]);
        doNewSend(["6", [31]]);
        doNewSend(["6", [27]]);
        doNewSend(["6", [10]]);
        doNewSend(["6", [38]]);
        doNewSend(["6", [28]]);
        doNewSend(["6", [25]]);
    }

    if (e.keyCode == 9 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        if (myPlayer.y < 2400){
            doNewSend(["13c", [0, 15, 0]]);
        } else if (myPlayer.y > 6850 && myPlayer.y < 7550){
            doNewSend(["13c", [0, 31, 0]]);
        } else {
	        doNewSend(["13c", [0, 12, 0]]);
        }
        doNewSend(["13c", [0, 11, 1]]);
    }

    if(e.keyCode == 82 && document.activeElement.id.toLowerCase() !== 'chatbox') {
            autoprimary = true;
            autosecondary = false;
            autoaim = true;
            autoprimary = true;
            autosecondary = false;
            doNewSend(["13c", [0, 0, 1]]);
            doNewSend(["5", [primary, true]]);
            doNewSend(["7", [1]]);
            doNewSend(["13c", [1, 7, 0]]);
            doNewSend(["13c", [0, 7, 0]]);
            doNewSend(["13c", [1, 21, 1]]);
            doNewSend(["13c", [0, 21, 1]]);
        setTimeout( () => {
            autoprimary = false;
            autosecondary = true;
            doNewSend(["13c", [0, 0, 0]]);
            doNewSend(["13c", [1, 53, 0]]);
            doNewSend(["13c", [0, 53, 0]]);
            doNewSend(["5", [secondary, true]]);
        }, 100);
        setTimeout( () => {
            doNewSend(["13c", [0, 0, 0]]);
            doNewSend(["7", [1]]);
            doNewSend(["5", [primary, true]]);
            doNewSend(["13c", [0, 11, 1]]);
            if (myPlayer.y < 2400){
                doNewSend(["13c", [0, 15, 0]]);
            } else if (myPlayer.y > 6850 && myPlayer.y < 7550){
                doNewSend(["13c", [0, 31, 0]]);
            } else {
	            doNewSend(["13c", [0, 12, 0]]);
            }
            autosecondary = false;
            autoaim = false;
        }, 215);
    }
})

document.addEventListener('keyup', (e)=>{
    spikePlacer.stop(e.keyCode);
    boostPlacer.stop(e.keyCode);
    boostSpiker.stop(e.keyCode);
    millPlacer.stop(e.keyCode);
    turretPlacer.stop(e.keyCode);
    healer1.stop(e.keyCode);
    healer2.stop(e.keyCode);
    if(e.keyCode == 71 && document.activeElement.id.toLowerCase() !== 'chatbox') {
        setTimeout( () => {
            doNewSend(["33", [null]]);
            boostDir = null;
        }, 10);
    }
})


function isElementVisible(e) {
    return (e.offsetParent !== null);
}

function fourSpawnpad() {
       place(spawnpadType, myPlayer.dir + toRad(135));
       place(spawnpadType, myPlayer.dir + toRad(150));
       place(spawnpadType, myPlayer.dir + toRad(165));
       place(spawnpadType, myPlayer.dir + toRad(180));
       place(spawnpadType, myPlayer.dir + toRad(270));
       place(spawnpadType, myPlayer.dir + toRad(360));
}

function toRad(angle) {
    return angle * 0.01745329251;
}

function dist(a, b){
    return Math.sqrt( Math.pow((b.y-a[2]), 2) + Math.pow((b.x-a[1]), 2) );
}

function update() {
    for (let i=0;i<9;i++){
        if (isElementVisible(document.getElementById("actionBarItem" + i.toString()))){
            primary = i;
        }
    }

    for (let i=9;i<16;i++){
        if (isElementVisible(document.getElementById("actionBarItem" + i.toString()))){
            secondary = i;
        }
    }

    for (let i=16;i<19;i++){
        if (isElementVisible(document.getElementById("actionBarItem" + i.toString()))){
            foodType = i - 16;
        }
    }

    for (let i=19;i<22;i++){
        if (isElementVisible(document.getElementById("actionBarItem" + i.toString()))){
            wallType = i - 16;
        }
    }

    for (let i=22;i<26;i++){
        if (isElementVisible(document.getElementById("actionBarItem" + i.toString()))){
            spikeType = i - 16;
        }
    }

    for (let i=26;i<29;i++){
        if (isElementVisible(document.getElementById("actionBarItem" + i.toString()))){
            millType = i - 16;
        }
    }

    for (let i=29;i<31;i++){
        if (isElementVisible(document.getElementById("actionBarItem" + i.toString()))){
            mineType = i - 16;
        }
    }

    for (let i=31;i<33;i++){
        if (isElementVisible(document.getElementById("actionBarItem" + i.toString()))){
            boostType = i - 16;
        }
    }

   for (let i=33;i<36;i++){
       if (isElementVisible(document.getElementById("actionBarItem" + i.toString()))){
           turretType = i - 16;
       }
   }

   for (let i=36;i<37;i++){
       if (isElementVisible(document.getElementById("actionBarItem" + i.toString()))){
           spawnpadType = i - 16;
       }
   }

   for (let i=37;i<39;i++){
       if (isElementVisible(document.getElementById("actionBarItem" + i.toString()))){
           turretType = i - 16;
       }
   }
}
var ezsound = new Audio("https://dl.dropboxusercontent.com/s/qjfmz3sxmig1rrp/Black%20Ops%202%20Kaboom%20Sound%20%28Nuketown%20Map%29.mp3?dl=0");

var kills = 10;

setInterval(getkills, 250);

function getkills(){
    var count = parseInt(document.getElementById("killCounter").innerText);
    if(count > kills){
	ezsound.play();
        doNewSend(["ch", ["        Get Baked.         "]]);
    }
    kills = count;
}
function spam(space, chance) {
    let result = '';
    let characters;
    if(space) {
        characters = 'You Like Cheese Cake?!';
    }
    if(space) {
        characters = characters.padStart((30 - characters.length) / 2 + characters.length)
        characters = characters.padEnd(30);
    }
    let count = 0;
    for (let i = 0; i < characters.length; i++ ) {
        if(Math.floor(Math.random() * chance) == 0 && characters.charAt(i) != "-" && count < 0 && characters.charAt(i) != " ") {
            result += "";
            count++
        } else {
            result += characters.charAt(i);
        }
    }
    return result;
}
$("#mapDisplay").css({background: `url('http://i.imgur.com/Qllo1mA.png')`});
document.getElementById('gameName').innerHTML = '🎂Cheese Cake🎂';
setInterval(() => {
    setTimeout(() => {
        document.getElementById("gameName").innerHTML = "🎂_heese Cake🎂";
        setTimeout(() => {
            document.getElementById("gameName").innerHTML = "🎂C_eese Cake🎂";
            setTimeout(() => {
                document.getElementById("gameName").innerHTML = "🎂Ch_ese Cake🎂";
                setTimeout(() => {
                    document.getElementById("gameName").innerHTML = "🎂Che_se Cake🎂";
                    setTimeout(() => {
                        document.getElementById("gameName").innerHTML = "🎂Chee_e Cake🎂";
                        setTimeout(() => {
                            document.getElementById("gameName").innerHTML = "🎂Chees_ Cake🎂";
                            setTimeout(() => {
                                document.getElementById("gameName").innerHTML = "🎂Cheese _ake🎂";
                                setTimeout(() => {
                                    document.getElementById("gameName").innerHTML = "🎂Cheese C_ke🎂";
                                    setTimeout(() => {
                                        document.getElementById("gameName").innerHTML = "🎂Cheese Ca_e🎂";
                                        setTimeout(() => {
                                            document.getElementById("gameName").innerHTML = "🎂Cheese Cak_🎂";
                                            setTimeout(() => {
                                                document.getElementById("gameName").innerHTML = "🎂Cheese_Cake🎂";
                                                setTimeout(() => {
                                                    document.getElementById("gameName").innerHTML = "🎂Cheese Cake🎂";
                                                }, 100);
                                            }, 100);
                                        }, 100);
                                    }, 100);
                                }, 100);
                            }, 100);
                        }, 100);
                    }, 100);
                }, 100);
            }, 100);
        }, 100);
    }, 100);
}, 300);
setInterval(() => {
    setTimeout(() => {
        document.getElementById('chatBox').placeholder = "🎂Message🎂";
        setTimeout(() => {
            document.getElementById('chatBox').placeholder = "🎂Message.🎂";
            setTimeout(() => {
                document.getElementById('chatBox').placeholder = "🎂Message..🎂";
                setTimeout(() => {
                    document.getElementById('chatBox').placeholder = "🎂Message...🎂";
                }, 100);
            }, 100);
        }, 100);
    }, 100);
}, 300)
setInterval(() => {
    setTimeout(() => {
        document.getElementById('nameInput').placeholder = "🎂Cheese Cake ;3🎂";
        setTimeout(() => {
            document.getElementById('nameInput').placeholder = "🎂_heese Cake ;3🎂";
            setTimeout(() => {
                document.getElementById('nameInput').placeholder = "🎂C_eese Cake ;3🎂";
                setTimeout(() => {
                    document.getElementById('nameInput').placeholder = "🎂Ch_ese Cake ;3🎂";
                    setTimeout(() => {
                        document.getElementById('nameInput').placeholder = "🎂Che_se Cake ;3🎂";
                        setTimeout(() => {
                            document.getElementById('nameInput').placeholder = "🎂Chee_e Cake ;3🎂";
                            setTimeout(() => {
                                document.getElementById('nameInput').placeholder = "🎂Chees_ Cake ;3🎂";
                                setTimeout(() => {
                                    document.getElementById('nameInput').placeholder = "🎂Cheese _ake ;3🎂";
                                    setTimeout(() => {
                                        document.getElementById('nameInput').placeholder = "🎂Cheese C_ke ;3🎂";
                                        setTimeout(() => {
                                            document.getElementById('nameInput').placeholder = "🎂Cheese Ca_e ;3🎂";
                                            setTimeout(() => {
                                                document.getElementById('nameInput').placeholder = "🎂Cheese Cak_ ;3🎂";
                                                setTimeout(() => {
                                                    document.getElementById('nameInput').placeholder = "🎂Cheese Cake ;3🎂";
                                                }, 100);
                                            }, 100);
                                        }, 100);
                                    }, 100);
                                }, 100);
                            }, 100);
                        }, 100);
                    }, 100);
                }, 100);
            }, 100);
        }, 100);
    }, 100);
}, 300);
document.getElementById('diedText').innerHTML = '🎂They Will Get Baked -_-🎂';
setInterval(() => {
    setTimeout(() => {
        document.getElementById('diedText').innerHTML = "🎂_hey Will Get Baked -_-🎂";
        setTimeout(() => {
            document.getElementById('diedText').innerHTML = "🎂T_ey Will Get Baked -_-🎂";
            setTimeout(() => {
                document.getElementById('diedText').innerHTML = "🎂Th_y Will Get Baked -_-🎂";
                setTimeout(() => {
                    document.getElementById('diedText').innerHTML = "🎂The_ Will Get Baked -_-🎂";
                    setTimeout(() => {
                        document.getElementById('diedText').innerHTML = "🎂They _ill Get Baked -_-🎂";
                        setTimeout(() => {
                            document.getElementById('diedText').innerHTML = "🎂They W_ll Get Baked -_-🎂";
                            setTimeout(() => {
                                document.getElementById('diedText').innerHTML = "🎂They Wi_l Get Baked -_-🎂";
                                setTimeout(() => {
                                    document.getElementById('diedText').innerHTML = "🎂They Wil_ Get Baked -_-🎂";
                                    setTimeout(() => {
                                        document.getElementById('diedText').innerHTML = "🎂They Will G_t Baked -_-🎂";
                                        setTimeout(() => {
                                            document.getElementById('diedText').innerHTML = "🎂They Will Get B_ked -_-🎂";
                                            setTimeout(() => {
                                                document.getElementById('diedText').innerHTML = "🎂They Will Get Ba_ed -_-🎂";
                                                setTimeout(() => {
                                                    document.getElementById('diedText').innerHTML = "🎂They Will Get Bake_ -_-🎂";
                                                }, 100);
                                            }, 100);
                                        }, 100);
                                    }, 100);
                                }, 100);
                            }, 100);
                        }, 100);
                    }, 100);
                }, 100);
            }, 100);
        }, 100);
    }, 100);
}, 300);
setInterval(() => {
setTimeout( () => {
document.getElementById('diedText').style.color = "orange";
setTimeout( () => {
document.getElementById('diedText').style.color = "pink";
setTimeout( () => {
document.getElementById('diedText').style.color = "purple";
setTimeout( () => {
document.getElementById('diedText').style.color = "orange";
setTimeout( () => {
document.getElementById('diedText').style.color = "pink";
setTimeout( () => {
document.getElementById('diedText').style.color = "purple";
setTimeout( () => {
document.getElementById('diedText').style.color = "orange";
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);}
 , 300);
setInterval(() => {
setTimeout( () => {
document.getElementById("pingDisplay").style.color = "orange";
setTimeout( () => {
document.getElementById("pingDisplay").style.color = "pink";
setTimeout( () => {
document.getElementById("pingDisplay").style.color = "purple";
setTimeout( () => {
document.getElementById("pingDisplay").style.color = "orange";
setTimeout( () => {
document.getElementById("pingDisplay").style.color = "pink";
setTimeout( () => {
document.getElementById("pingDisplay").style.color = "purple";
setTimeout( () => {
document.getElementById("pingDisplay").style.color = "orange";
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);}
 , 300);
setInterval(() => {
setTimeout( () => {
document.getElementById("ytLink").style.color = "orange";
setTimeout( () => {
document.getElementById("ytLink").style.color = "pink";
setTimeout( () => {
document.getElementById("ytLink").style.color = "purple";
setTimeout( () => {
document.getElementById("ytLink").style.color = "orange";
setTimeout( () => {
document.getElementById("ytLink").style.color = "pink";
setTimeout( () => {
document.getElementById("ytLink").style.color = "purple";
setTimeout( () => {
document.getElementById("ytLink").style.color = "orange";
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);}
 , 300);
setInterval(() => {
setTimeout( () => {
document.getElementById("serverBrowser").style.color = "orange";
setTimeout( () => {
document.getElementById("serverBrowser").style.color = "pink";
setTimeout( () => {
document.getElementById("serverBrowser").style.color = "purple";
setTimeout( () => {
document.getElementById("serverBrowser").style.color = "orange";
setTimeout( () => {
document.getElementById("serverBrowser").style.color = "pink";
setTimeout( () => {
document.getElementById("serverBrowser").style.color = "purple";
setTimeout( () => {
document.getElementById("serverBrowser").style.color = "orange";
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);}
 , 300);
setInterval(() => {
    setTimeout(() => {
        document.getElementById('enterGame').innerHTML = "🎂Go Bake!🎂";
        setTimeout(() => {
            document.getElementById('enterGame').innerHTML = "🎂_o Bake!🎂";
            setTimeout(() => {
                document.getElementById('enterGame').innerHTML = "🎂Go _ake!🎂";
                setTimeout(() => {
                    document.getElementById('enterGame').innerHTML = "🎂Go B_ke!🎂";
                    setTimeout(() => {
                        document.getElementById('enterGame').innerHTML = "🎂Go Ba_e!🎂";
                        setTimeout(() => {
                            document.getElementById('enterGame').innerHTML = "🎂Go Bak_!🎂";
                            setTimeout(() => {
                                document.getElementById('enterGame').innerHTML = "🎂_o Bake!🎂";
                                setTimeout(() => {
                                    document.getElementById('enterGame').innerHTML = "🎂G_ Bake!🎂";
                                    setTimeout(() => {
                                        document.getElementById('enterGame').innerHTML = "🎂Go _ake!🎂";
                                        setTimeout(() => {
                                            document.getElementById('enterGame').innerHTML = "🎂Go B_ke!🎂";
                                            setTimeout(() => {
                                                document.getElementById('enterGame').innerHTML = "🎂Go Ba_e!🎂";
                                                setTimeout(() => {
                                                    document.getElementById('enterGame').innerHTML = "🎂Go Bak_!🎂";
                                                }, 100);
                                            }, 100);
                                        }, 100);
                                    }, 100);
                                }, 100);
                            }, 100);
                        }, 100);
                    }, 100);
                }, 100);
            }, 100);
        }, 100);
    }, 100);
}, 200);
document.getElementById("storeHolder").style = "height: 2200px; width: 500px;"
document.getElementById("allianceHolder").style = "height: 2200px; width: 500px;"

document.getElementById('adCard').remove();
document.getElementById('errorNotification').remove();
setInterval(() => {
setTimeout( () => {
document.getElementById("leaderboard").style.color = "orange";
setTimeout( () => {
document.getElementById("leaderboard").style.color = "pink";
setTimeout( () => {
document.getElementById("leaderboard").style.color = "purple";
setTimeout( () => {
document.getElementById("leaderboard").style.color = "orange";
setTimeout( () => {
document.getElementById("leaderboard").style.color = "pink";
setTimeout( () => {
document.getElementById("leaderboard").style.color = "purple";
setTimeout( () => {
document.getElementById("leaderboard").style.color = "orange";
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);}
 , 300);
document.getElementById("promoImg").remove();
document.getElementById("nameInput").style.color = "orange"
setTimeout( () => {
document.getElementById("nameInput").style.color = "pink"
}, 2000);
setInterval(() => {
setTimeout( () => {
document.getElementById("woodDisplay").style.color = "orange";
setTimeout( () => {
document.getElementById("woodDisplay").style.color = "pink";
setTimeout( () => {
document.getElementById("woodDisplay").style.color = "purple";
setTimeout( () => {
document.getElementById("woodDisplay").style.color = "orange";
setTimeout( () => {
document.getElementById("woodDisplay").style.color = "pink";
setTimeout( () => {
document.getElementById("woodDisplay").style.color = "purple";
setTimeout( () => {
document.getElementById("woodDisplay").style.color = "orange";
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);}
 , 300);
setInterval(() => {
setTimeout( () => {
document.getElementById("scoreDisplay").style.color = "orange";
setTimeout( () => {
document.getElementById("scoreDisplay").style.color = "pink";
setTimeout( () => {
document.getElementById("scoreDisplay").style.color = "purple";
setTimeout( () => {
document.getElementById("scoreDisplay").style.color = "orange";
setTimeout( () => {
document.getElementById("scoreDisplay").style.color = "pink";
setTimeout( () => {
document.getElementById("scoreDisplay").style.color = "purple";
setTimeout( () => {
document.getElementById("scoreDisplay").style.color = "orange";
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);}
 , 300);
setInterval(() => {
setTimeout( () => {
document.getElementById("stoneDisplay").style.color = "orange";
setTimeout( () => {
document.getElementById("stoneDisplay").style.color = "pink";
setTimeout( () => {
document.getElementById("stoneDisplay").style.color = "purple";
setTimeout( () => {
document.getElementById("stoneDisplay").style.color = "orange";
setTimeout( () => {
document.getElementById("stoneDisplay").style.color = "pink";
setTimeout( () => {
document.getElementById("stoneDisplay").style.color = "purple";
setTimeout( () => {
document.getElementById("stoneDisplay").style.color = "orange";
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);}
 , 300);
setInterval(() => {
setTimeout( () => {
document.getElementById("foodDisplay").style.color = "orange";
setTimeout( () => {
document.getElementById("foodDisplay").style.color = "pink";
setTimeout( () => {
document.getElementById("foodDisplay").style.color = "purple";
setTimeout( () => {
document.getElementById("foodDisplay").style.color = "orange";
setTimeout( () => {
document.getElementById("foodDisplay").style.color = "pink";
setTimeout( () => {
document.getElementById("foodDisplay").style.color = "purple";
setTimeout( () => {
document.getElementById("foodDisplay").style.color = "orange";
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);}
 , 300);
setInterval(() => {
setTimeout( () => {
document.getElementById("killCounter").style.color = "orange";
setTimeout( () => {
document.getElementById("killCounter").style.color = "pink";
setTimeout( () => {
document.getElementById("killCounter").style.color = "purple";
setTimeout( () => {
document.getElementById("killCounter").style.color = "orange";
setTimeout( () => {
document.getElementById("killCounter").style.color = "pink";
setTimeout( () => {
document.getElementById("killCounter").style.color = "purple";
setTimeout( () => {
document.getElementById("killCounter").style.color = "orange";
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);}
 , 300);
setInterval(() => {
setTimeout( () => {
document.getElementById("ageText").style.color = "orange";
setTimeout( () => {
document.getElementById("ageText").style.color = "pink";
setTimeout( () => {
document.getElementById("ageText").style.color = "purple";
setTimeout( () => {
document.getElementById("ageText").style.color = "orange";
setTimeout( () => {
document.getElementById("ageText").style.color = "pink";
setTimeout( () => {
document.getElementById("ageText").style.color = "purple";
setTimeout( () => {
document.getElementById("ageText").style.color = "orange";
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);}
 , 300);
setInterval(() => {
setTimeout( () => {
document.getElementById("allianceButton").style.color = "orange";
setTimeout( () => {
document.getElementById("allianceButton").style.color = "pink";
setTimeout( () => {
document.getElementById("allianceButton").style.color = "purple";
setTimeout( () => {
document.getElementById("allianceButton").style.color = "orange";
setTimeout( () => {
document.getElementById("allianceButton").style.color = "pink";
setTimeout( () => {
document.getElementById("allianceButton").style.color = "purple";
setTimeout( () => {
document.getElementById("allianceButton").style.color = "orange";
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);}
 , 300);
setInterval(() => {
setTimeout( () => {
document.getElementById("chatButton").style.color = "orange";
setTimeout( () => {
document.getElementById("chatButton").style.color = "pink";
setTimeout( () => {
document.getElementById("chatButton").style.color = "purple";
setTimeout( () => {
document.getElementById("chatButton").style.color = "orange";
setTimeout( () => {
document.getElementById("chatButton").style.color = "pink";
setTimeout( () => {
document.getElementById("chatButton").style.color = "purple";
setTimeout( () => {
document.getElementById("chatButton").style.color = "orange";
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);}
 , 300);
setInterval(() => {
setTimeout( () => {
document.getElementById("storeButton").style.color = "orange";
setTimeout( () => {
document.getElementById("storeButton").style.color = "pink";
setTimeout( () => {
document.getElementById("storeButton").style.color = "purple";
setTimeout( () => {
document.getElementById("storeButton").style.color = "orange";
setTimeout( () => {
document.getElementById("storeButton").style.color = "pink";
setTimeout( () => {
document.getElementById("storeButton").style.color = "purple";
setTimeout( () => {
document.getElementById("storeButton").style.color = "orange";
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);}
 , 300);
setInterval(() => {
setTimeout( () => {
document.getElementById("gameName").style.color = "orange";
setTimeout( () => {
document.getElementById("gameName").style.color = "pink";
setTimeout( () => {
document.getElementById("gameName").style.color = "purple";
setTimeout( () => {
document.getElementById("gameName").style.color = "orange";
setTimeout( () => {
document.getElementById("gameName").style.color = "pink";
setTimeout( () => {
document.getElementById("gameName").style.color = "purple";
setTimeout( () => {
document.getElementById("gameName").style.color = "orange";
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);}
 , 300);
setInterval(() => {
setTimeout( () => {
document.getElementById('loadingText').style.color = "orange";
setTimeout( () => {
document.getElementById('loadingText').style.color = "pink";
setTimeout( () => {
document.getElementById('loadingText').style.color = "purple";
setTimeout( () => {
document.getElementById('loadingText').style.color = "orange";
setTimeout( () => {
document.getElementById('loadingText').style.color = "pink";
setTimeout( () => {
document.getElementById('loadingText').style.color = "purple";
setTimeout( () => {
document.getElementById('loadingText').style.color = "orange";
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);}
 , 300);
document.getElementById('loadingText').innerHTML = '....🎂Loading🎂Cheese🎂Cake🎂....';
setTimeout( () => {
document.getElementById('loadingText').innerHTML = '✅Loading Succes!✅';
}, 2000)
setInterval(() => {
setTimeout( () => {
document.getElementById("setupCard").style.color = "orange";
setTimeout( () => {
document.getElementById("setupCard").style.color = "pink";
setTimeout( () => {
document.getElementById("setupCard").style.color = "purple";
setTimeout( () => {
document.getElementById("setupCard").style.color = "orange";
setTimeout( () => {
document.getElementById("setupCard").style.color = "pink";
setTimeout( () => {
document.getElementById("setupCard").style.color = "purple";
setTimeout( () => {
document.getElementById("setupCard").style.color = "orange";
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);}
 , 300);
setInterval(() => {
setTimeout( () => {
document.getElementById("joinPartyButton").style.color = "orange";
setTimeout( () => {
document.getElementById("joinPartyButton").style.color = "pink";
setTimeout( () => {
document.getElementById("joinPartyButton").style.color = "purple";
setTimeout( () => {
document.getElementById("joinPartyButton").style.color = "orange";
setTimeout( () => {
document.getElementById("joinPartyButton").style.color = "pink";
setTimeout( () => {
document.getElementById("joinPartyButton").style.color = "purple";
setTimeout( () => {
document.getElementById("joinPartyButton").style.color = "orange";
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);}
 , 300);
setInterval(() => {
setTimeout( () => {
document.getElementById("setupCard").style.backgroundColor = "orange";
setTimeout( () => {
document.getElementById("setupCard").style.backgroundColor = "pink";
setTimeout( () => {
document.getElementById("setupCard").style.backgroundColor = "purple";
setTimeout( () => {
document.getElementById("setupCard").style.backgroundColor = "orange";
setTimeout( () => {
document.getElementById("setupCard").style.backgroundColor = "pink";
setTimeout( () => {
document.getElementById("setupCard").style.backgroundColor = "purple";
setTimeout( () => {
document.getElementById("setupCard").style.backgroundColor = "orange";
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);}
 , 300);
setInterval(() => {
setTimeout( () => {
document.getElementById("guideCard").style.backgroundColor = "orange";
setTimeout( () => {
document.getElementById("guideCard").style.backgroundColor = "pink";
setTimeout( () => {
document.getElementById("guideCard").style.backgroundColor = "purple";
setTimeout( () => {
document.getElementById("guideCard").style.backgroundColor = "orange";
setTimeout( () => {
document.getElementById("guideCard").style.backgroundColor = "pink";
setTimeout( () => {
document.getElementById("guideCard").style.backgroundColor = "purple";
setTimeout( () => {
document.getElementById("guideCard").style.backgroundColor = "orange";
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);}
 , 300);
setInterval(() => {
setTimeout( () => {
document.getElementById("enterGame").style.backgroundColor = "orange";
setTimeout( () => {
document.getElementById("enterGame").style.backgroundColor = "pink";
setTimeout( () => {
document.getElementById("enterGame").style.backgroundColor = "purple";
setTimeout( () => {
document.getElementById("enterGame").style.backgroundColor = "orange";
setTimeout( () => {
document.getElementById("enterGame").style.backgroundColor = "pink";
setTimeout( () => {
document.getElementById("enterGame").style.backgroundColor = "purple";
setTimeout( () => {
document.getElementById("enterGame").style.backgroundColor = "orange";
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);
}, 100);}
 , 300);
