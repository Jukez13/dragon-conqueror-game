<template>
    <div class="wrapper">
        <h1>{{ title }}</h1>
        <div class="container">
            <div class="row">
                <div class="col">
                    <div class="health">
                        <div class="health-bar" :style="warriorHealthBar">
                            <span class="hp-percent">{{ warriorHealth }} %</span>
                        </div>
                    </div>
                    <p class="text-center">Warrior</p>
                    <div class="img-container">
                        <img src="./images/warrior.gif" alt="warrior" />
                    </div>
                </div>
                <div class="col">
                    <div class="health">
                        <div class="health-bar" :style="dragonHealthBar">
                            <span class="hp-percent">{{ dragonHealth }} %</span>
                        </div>
                    </div>
                    <p class="text-center">Dragon</p>
                    <div class="img-container">
                        <img src="./images/dragon.gif" alt="dragon" />
                    </div>
                </div>
            </div>
            <div class="row">
                <div class="crow">
                    <button class="control-icon attack" @click="attackDragon">
                        <img src="./images/attack_icon.jpg" alt="attack" />
                    </button>
                    <button class="control-icon" @click="specialAttack" :disabled="numOfSpecialAttack === 0">
                        <img src="./images/special_attack.jpg" alt="special_attack" />
                        <span class="num-of-attempt">{{ numOfSpecialAttack }}</span>
                    </button>
                    <button class="control-icon" @click="heal" :disabled="numOfHeal === 0">
                        <img src="./images/heal_icon.webp" alt="heal" /><span class="num-of-attempt">{{
                            numOfHeal
                        }}</span>
                    </button>
                    <button class="control-icon" @click="forfeit">
                        <img src="./images/surrender_icon.png" alt="give-up" />
                    </button>
                </div>
            </div>
            <div class="row mt-5">
                <div class="container">
                    <h4 class="text-center">Battle Logs</h4>
                    <ul class="col-md-5 logs">
                        <li v-for="(logMessage, index) in battleLogMessages" :key="index" class="text-center">
                            <span :class="{
                                warrior: logMessage.attacker === 'warrior',
                                dragon: logMessage.attacker === 'dragon'
                            }">
                                <!-- {{ logMessage.attacker === 'warrior' ? `<img src='./gifs/warrior1.gif' alt="dragon">` : 'dragon'}} -->
                                <img src="./images/warrior.gif" alt="warrior" v-if="logMessage.attacker === 'warrior'"
                                    class="img-icon" />
                                <img src="./images/dragon.gif" alt="warrior" v-else class="img-icon" />
                            </span>
                            <span v-if="logMessage.actionType === 'heal'">
                                heals himself for
                                <span class="log-heal">{{ logMessage.actionValue }} hp</span>
                            </span>
                            <span v-else-if="logMessage.actionType === 'attack'">
                                attack and deals
                                <span class="log-damage">{{ logMessage.actionValue }} of damage</span>
                            </span>
                            <span v-else-if="logMessage.actionType === 'special-attack'">
                                use special attack and deals
                                <span class="log--damage">{{ logMessage.actionValue }} of damage</span>
                            </span>
                        </li>
                    </ul>
                </div>
            </div>
        </div>
        <div class="overlap" v-if="winner">
            <div class="winner">
                <div class="text-center">
                    <h3>Game Over!</h3>
                    <h4 v-if="winner === 'dragon'">You Lose!</h4>
                    <h4 v-else-if="winner === 'warrior'">You Win!</h4>
                    <h4 v-else-if="winner === 'draw'">It's a Draw!</h4>
                    <button @click="startFight" class="btn start-btn mt-4">
                        Battle Again
                    </button>
                </div>
            </div>
        </div>
    </div>
</template>


<script>
const getRandomValue = (min, max) => {
    return Math.floor(Math.random() * (max - min)) + min;
};
export default {
    name: "GameComponent",
    data() {
        return {
            title: 'Dragon and Conqueror Game!',
            warriorHealth: 100,
            dragonHealth: 100,
            numOfSpecialAttack: 3,
            numOfHeal: 3,
            winner: null,
            battleLogMessages: [],
            swordSlash: new Audio(require("./audios/attack.wav")),
            dragonBreath: new Audio(require("./audios/dragon_roar.mp3")),
            healEffect: new Audio(require("./audios/heal.mp3")),
            specialAttackEffect: new Audio(
                require("./audios/special_attack.mp3")
            )
        };
    },
    watch: {
        warriorHealth(value) {
            if (value <= 0 && this.dragonHealth <= 0) {
                //draw
                this.winner = "draw";
            } else if (value <= 0) {
                this.winner = "dragon";
            }
        },
        dragonHealth(value) {
            if (value <= 0 && this.warriorHealth <= 0) {
                //draw
                this.winner = "draw";
            } else if (value <= 0) {
                this.winner = "warrior";
            }
        }
    },
    computed: {
        warriorHealthBar() {
            if (this.warriorHealth < 0) {
                return { width: "0%" };
            } else {
                if (this.warriorHealth <= 50) {
                    return { width: this.warriorHealth + "%", background: "red" };
                } else {
                    return { width: this.warriorHealth + "%" };
                }
            }
        },
        dragonHealthBar() {
            if (this.dragonHealth < 0) {
                return { width: "0%" };
            } else {
                if (this.dragonHealth <= 50) {
                    return { width: this.dragonHealth + "%", background: "red" };
                } else {
                    return { width: this.dragonHealth + "%" };
                }
            }
        }
    },
    methods: {
        startFight() {
            this.warriorHealth = 100;
            this.dragonHealth = 100;
            this.numOfSpecialAttack = 3;
            this.numOfHeal = 3;
            this.winner = null;
            this.battleLogMessages = [];
        },
        attackDragon() {
            const attackDamage = getRandomValue(6, 15);
            // check if dragon health is greater than zero first
            if (this.dragonHealth - attackDamage < 0) {
                this.dragonHealth = 0;
            } else {
                this.dragonHealth -= attackDamage;
            }
            this.swordSlash.play();
            console.log(this.swordSlash);
            this.addBattleLog("warrior", "attack", attackDamage);
            this.attackWarrior();
        },
        attackWarrior() {
            const attackDamage = getRandomValue(8, 20);
            if (this.warriorHealth - attackDamage < 0) {
                this.warriorHealth = 0;
            } else {
                this.warriorHealth -= attackDamage;
            }
            setTimeout(() => {
                this.dragonBreath.play();
                setTimeout(() => {
                    this.dragonBreath.pause();
                }, 2000);
            }, 0);

            this.addBattleLog("dragon", "attack", attackDamage);
        },
        specialAttack() {
            this.numOfSpecialAttack--;
            const attackDamage = getRandomValue(10, 25);
            if (this.dragonHealth - attackDamage < 0) {
                this.dragonHealth = 0;
            } else {
                this.specialAttackEffect.play();
                this.dragonHealth -= attackDamage;
            }
            this.addBattleLog("warrior", "special-attack", attackDamage);
            this.attackWarrior();
        },
        heal() {
            this.numOfHeal--;
            const healValue = getRandomValue(10, 25);
            if (this.warriorHealth + healValue > 100) {
                this.warriorHealth = 100;
            } else {
                this.warriorHealth += healValue;
            }
            this.healEffect.play();
            this.addBattleLog("warrior", "heal", healValue);
            this.attackWarrior();
        },
        forfeit() {
            this.winner = "dragon";
        },
        addBattleLog(who, what, value) {
            this.battleLogMessages.unshift({
                attacker: who,
                actionType: what,
                actionValue: value
            });
        }
    }
};
</script>

<style scoped>
h1 {
    text-align: center;
    margin-bottom: 20px;
}

* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    font-family: "Jost", sans-serif;
    background-color: #03071E;
    color: #fff;
}

.wrapper {
    width: 100%;
    min-height: 100vh;
    position: relative;
}

.col {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    width: 100%;
}

.row {
    display: flex;
    justify-content: space-evenly;
    width: 100%;
    align-items: center;
}

.crow {
    display: flex;
    flex-direction: row;
    justify-content: center;
    width: 100%;
    align-items: center;
}

.img-container {
    margin: 0 auto;
    width: 60%;
}

.img-container img {
    width: 100%;
    height: 100%;
}

.health {
    margin: 0 auto;
    width: 80%;
    border: 1px solid #fff;
    border-radius: 3px;
    height: 25px;
    position: relative;
}

.health .health-bar {
    width: 100%;
    height: 100%;
    background: #477d48;
    border-radius: 3px;
    display: flex;
    justify-content: center;
    align-items: center;
    transition: 0.2s ease;
}

.hp-percent {
    position: absolute;
    left: 50%;
    font-size: 0.8rem;
    color: #fff;
}

/*CONTROLS*/
.controls {
    display: flex;
    justify-content: center;
}

.control-icon {
    width: 30px;
    height: 30px;
    margin-right: 0.5rem;
    padding: 2px;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #9d0208;
    border-radius: 2px;
    transition: 300ms ease-out;
    position: relative;
}

.control-icon img {
    width: 100%;
    height: 100%;
    background-color: #e85d04;
}

.control-icon img:hover {
    background: #faa307;
}

.num-of-attempt {
    position: absolute;
    height: 1rem;
    display: flex;
    align-items: center;
    justify-content: center;
    background: #dc2f02;
    padding: 3px;
    color: #fff;
    top: -30%;
    right: -20%;
    border-radius: 5px;
}

.start-btn {
    background: #ffba08;
    color: #03071e;
    border-radius: 5px;
    padding: 5px;
}

.start-btn:hover {
    background: #f48c06 !important;
    color: #370617;
}

.logs {
    margin: 0 auto;
    padding: 1rem;
    border-radius: 0.3rem;
    background: #370617;
    color: #fff;
}

.logs li {
    list-style-type: none;
}

.warrior {
    color: #dc2f02;
}

.dragon {
    color: #e85d04;
}

.img-icon {
    width: 50px;
}

.log-heal {
    color: #477d48;
}

.log-damage {
    color: #faa307;
}

button:disabled,
button[disabled] {
    /* border: 1px solid #999999!important; */
    background-color: #cccccc !important;
    color: #666666 !important;
    border-radius: 2px;
}

button:disabled,
button[disabled] img {
    background: #cccccc;
}

button:disabled,
button[disabled] img:hover {
    background: #f4f4f4;
}

/*Game Over Display*/
.overlap {
    position: absolute;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    background: rgb(19, 18, 18, 0.95);
    z-index: 100;
}

.overlap .winner {
    background: #dc2f02;
    color: #fff;
    width: 300px;
    height: 300px;
    border-radius: 10px;
    margin: auto;
    margin-top: 20%;
    display: flex;
    align-items: center;
    justify-content: center;
}
</style>