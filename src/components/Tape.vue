<template>
  <div class="tape-block">
    <div class="tape-container">
        <button id="back" @click="moveBack"><img src="../assets/arrow.png" alt=""></button>
        <div class="tape-wrapper">
            <div ref="tapeDiv" class="tape">
            <div class="cell" v-for="(cell, i) of tape" :key="i">
                <p class="cell-id">{{ cell.id }}</p>
                <input type="text" class="cell-symbol" v-model="cell.symbol" maxlength="1" @input="validate(cell.id)">
            </div>
        </div>
        </div>
        <button id="forward" @click="moveForward"><img src="../assets/arrow.png" alt=""></button>
    </div>
    <div class="alphabet-wrapper">
        <div class="controls">
            <div class="alphabet">
                <p>Alphabet</p>
                <input type="text" id="alphabet" v-model="alphabet" @input="validateAlphabet()">
            </div>
            <div class="play">
                <button @click="play(1)">
                    <svg stroke="white" width="25px" height="25px" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M16.6582 9.28638C18.098 10.1862 18.8178 10.6361 19.0647 11.2122C19.2803 11.7152 19.2803 12.2847 19.0647 12.7878C18.8178 13.3638 18.098 13.8137 16.6582 14.7136L9.896 18.94C8.29805 19.9387 7.49907 20.4381 6.83973 20.385C6.26501 20.3388 5.73818 20.0469 5.3944 19.584C5 19.053 5 18.1108 5 16.2264V7.77357C5 5.88919 5 4.94701 5.3944 4.41598C5.73818 3.9531 6.26501 3.66111 6.83973 3.6149C7.49907 3.5619 8.29805 4.06126 9.896 5.05998L16.6582 9.28638Z" stroke="#000000" stroke-width="2" stroke-linejoin="round"/></svg>
                </button>
                <button @click="stop">
                    <svg stroke="white" width="25px" height="25px" viewBox="-0.5 0 25 25" fill="none" xmlns="http://www.w3.org/2000/svg">
                        <path d="M10 6.42004C10 4.76319 8.65685 3.42004 7 3.42004C5.34315 3.42004 4 4.76319 4 6.42004V18.42C4 20.0769 5.34315 21.42 7 21.42C8.65685 21.42 10 20.0769 10 18.42V6.42004Z" stroke="#000000" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                        <path d="M20 6.42004C20 4.76319 18.6569 3.42004 17 3.42004C15.3431 3.42004 14 4.76319 14 6.42004V18.42C14 20.0769 15.3431 21.42 17 21.42C18.6569 21.42 20 20.0769 20 18.42V6.42004Z" stroke="#000000" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                    </svg>
                </button>
                <button @click="step">
                    <svg stroke="white" width="25px" height="25px" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                        <path d="M20 4V20M4 12H16M16 12L12 8M16 12L12 16" stroke="#000000" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                    </svg>
                </button>
                <button @click="play(2)">
                    <p>x2</p>
                </button>      
                <button @click="play(4)">
                    <p>x4</p>
                </button> 
                <button @click="play(8)">
                    <p>x8</p>
                </button>                     
            </div>
        </div>
    </div>
    <div class="states-wrapper">
        <div class="states">
            <div class="states-header">
                <p>States</p>
                <button @click="states.push([])">+</button>
            </div>
            <table class="states-table">
                <tr class="header-row">
                    <th></th>
                    <th v-for="(state, i) of states" :key="i">
                        <div class="state-header">
                            <p>Q{{ i+1 }}</p>
                            <button @click="states.splice(i, 1)">-</button>
                        </div>
                    </th>
                </tr>
                <tr v-for="(letter, i) of alphabet" :key="i">
                    <th>{{ letter }}</th>
                    <td v-for="(state, j) of states" :key="j">
                        <input type="text" v-model="states[j][i]" @focusout="validateState(j,i)">
                    </td>
                </tr>
            </table>
        </div>
    </div>
  </div>
</template>
<script setup>

import { ref, onMounted } from 'vue'

const tapeDiv = ref(null)
const tape = ref([])    
const alphabet = ref('_')
const states = ref([[]])
const pick = ref(0)
const currentState = ref(1)
for (let i = -55; i <= 55; i++) {
    tape.value.push({ id: tape.value.length-55, symbol: '_' })
}

onMounted(() => {
    tapeDiv.value.style.marginLeft='0px'
})
let interval
const play = (speed)=>{
    clearInterval(interval)
    interval = setInterval(() => {
        step()
    }, 1000/speed)
}

const stop = ()=>{
    clearInterval(interval)
}

const step = ()=>{
    let tempSymbol = tape.value[pick.value+Math.floor(tape.value.length/2)].symbol
    let tempState = states.value[currentState.value-1][alphabet.value.indexOf(tape.value[pick.value+Math.floor(tape.value.length/2)].symbol)]
    tape.value[pick.value+Math.floor(tape.value.length/2)].symbol = states.value[currentState.value-1][alphabet.value.indexOf(tape.value[pick.value+Math.floor(tape.value.length/2)].symbol)][0]
    console.log(states.value[currentState.value-1][alphabet.value.indexOf(tempSymbol)].substr(3))
    currentState.value = parseInt(states.value[currentState.value-1][alphabet.value.indexOf(tempSymbol)].substr(3))
    if (tempState.substr(3) == '0'){
        stop()
        currentState.value=1
        setTimeout(() => {
            alert("Done")
        }, 10);
        return
    }
    if (tempState[1] === '>'){
        moveForward()
    } else if (tempState[1] === '<'){
        moveBack()
    } else if (tempState[1] === '.'){
        return
    }
}

const moveBack = () => {
    tapeDiv.value.style.marginLeft = 0+parseInt(tapeDiv.value.style.marginLeft,10)+50+'px'
    pick.value--
    if (Math.abs(Math.abs(pick.value)-tape.value.length/2)<25) {
        console.log('hi')
        tape.value.splice(0, 0, { id: tape.value[0].id-1, symbol: '_' })
        tape.value.push({ id: tape.value[tape.value.length-1].id+1, symbol: '_' }) 
    }
}
const moveForward = () => {
    tapeDiv.value.style.marginLeft = 0+parseInt(tapeDiv.value.style.marginLeft,10)-50+'px'
    pick.value++
    if (Math.abs(Math.abs(pick.value)-tape.value.length/2)<25) {
        console.log('hi')
        tape.value.splice(0, 0, { id: tape.value[0].id-1, symbol: '_' })
        tape.value.push({ id: tape.value[tape.value.length-1].id+1, symbol: '_' }) 
    }
}

const validate = (i)=>{
    if (!alphabet.value.includes(tape.value[i+Math.floor(tape.value.length/2)].symbol)){
        tape.value[i+Math.floor(tape.value.length/2)] = { id: i, symbol: '' }
    }
}

const validateAlphabet = ()=>{
    alphabet.value = String.prototype.concat.call(...new Set(alphabet.value))
}

const validateState = (j, i)=>{
    let reg = new RegExp(`[${alphabet.value}]{1}[><.]Q?\\d{1,3}`)
    if (!reg.test(states.value[j][i])){
        states.value[j][i] = ''
    } else{
        if (states.value[j][i][2] =='Q'){
            return
        }
        states.value[j][i] = states.value[j][i].substr(0,2) + 'Q'+states.value[j][i].substr(2)
    }
}

window.addEventListener("keydown", (event) => {
    if (document.activeElement.tagName === 'INPUT'){
        return
    }
    if (alphabet.value.includes(event.key)) {
        tape.value[pick.value+Math.floor(tape.value.length/2)].symbol = event.key
    }
    if (event.key === 'ArrowRight') {
        moveForward()
    } else if (event.key === 'ArrowLeft') {
        moveBack()
    }
})


</script>

<style scoped>

.tape-block{
    display: flex;
    flex-direction: column;
    gap: 25px;
}

.tape-container{
    display: flex;
    width: 100%;
    justify-content: center;
    height: 100px;
    padding: 20px 0;
    /* border-top: 2px solid var(--main-color);
    border-bottom: 2px solid var(--main-color); */
}
button{
    display: flex;
    justify-content: center;
    align-items: center;
    border: none;
    background: linear-gradient(var(--accent-color), var(--secondary-color));
    height: 35px;
    width: 35px;
    border-radius: 5px;
    font-size: 18px;
    font-weight: bold;
    color: white;
    transition: transform 0.3s;
}
button:hover{
    transform: scale(1.1);
}
button>p{
    color: white;
}
svg *{
    stroke: white;
}
.tape-container button{
    height: 100%;
    background: transparent;
    border: none;
    border-radius: 5px;
    padding: 5px;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: transform 0.3s;
}
.tape-container button:hover{
    transform: scale(1.2);
}
.tape-container button#back{
    rotate: 180deg;
}
.tape-container button img{
    width: 15px;
}

.tape-wrapper{
    display: flex;
    justify-content: center;
    overflow: hidden;
    max-width: 1225px;
    border: 2px solid var(--main-color);
    border-bottom: 0;
    border-radius: 5px;
}
.tape-wrapper::before{
    content: '';
    position: absolute;
    height: 25px;
    width: 25px;
    border: 2px solid var(--accent-color);
    border-radius: 5px;
}
.tape{
    display: flex;
    height: 100%;
    position: relative;
    transition: margin-left 0.3s;
}
.cell{
    height: 100%;
    width: 25px;
}
.cell *{
    width: 25px;
    display: flex;
    align-items: center;
    justify-content: center;
}
.cell-id{
    height: 50%;
    font-size: 10px;
}
.cell-symbol{
    border:2px solid var(--main-color);
    border-left: none;
    height: 50%;
    position: relative;
    text-align: center;
}

.alphabet-wrapper{
    display: flex;
    justify-content: center;
}
.controls{
    max-width: 1225px;
    width: 100%;
    display: flex;
    align-items: center;
    gap: 50px;
}
.alphabet input{
    border: 2px solid var(--main-color);
    border-radius: 5px;
    padding: 5px;
    font-weight: bold;
    width: 100%;
    max-width: 250px;
}
.play{
    display: flex;
    gap: 10px
}

.states-wrapper{
    display: flex;
    justify-content: center;
}
.states{
    max-width: 1225px;
    width: 100%;
    display: flex;
    flex-direction: column;
    gap: 10px;
}
.states *{
    width: fit-content;
} 
.states-header{
    display: flex;
    align-items: center;
    gap: 15px;
}
.states-header button{
    border: none;
    background: linear-gradient(var(--accent-color), var(--secondary-color));
    height: 25px;
    width: 25px;
    border-radius: 5px;
    font-size: 18px;
    font-weight: bold;
    color: white;
    transition: transform 0.3s;
}
.states-header button:hover{
    transform: scale(1.1);
}
.states-table{
    border: 2px solid var(--main-color);
    border-collapse: collapse;
}
.states-table th, td{
    border: 2px solid var(--main-color);
    padding: 5px 10px;
    min-width: 40px;
}

.states-table td input{
    text-align: center;

}
.state-header{
    display: flex;
}

.states-table th button{
    border: none;
    background: transparent;
    color: black;
    height: auto;
    width: auto;
    font-size: 18px;
    font-weight: bold;
    transition: transform 0.3s;
}
.states-table th button:hover{
    transform: scale(1.2);
}
.states-table td{
    padding: 0;
}
.states-table input{
    border: none;
    width: 50px;
    height: 100%;
    padding: 5px;
}

</style>
