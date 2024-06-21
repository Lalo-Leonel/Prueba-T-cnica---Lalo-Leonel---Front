<template>
    <div class="" style="display: flex; flex-wrap: wrap; justify-content: space-between; align-items: center;">
        <div>
            <div class="text">USUARIO: {{ userGame.name }}</div>
            <div class="text">SALDO: {{ userGame.balance.toFixed(2) }}</div>
        </div>
        <div>
            <button class="primary-btn" @click="showModal = true">Registrarse</button>
            <Modal :visible="showModal" @close="showModal = false">
                <div class="modal-content">
                    <h2 class="modal-title">NUEVO USUARIO</h2>
                    <div class="modal-body">
                        <div class="modal-input">
                            <label>nombre</label>
                            <input type="text" v-model="inputUser.name">
                        </div>
                        <div class="modal-input">
                            <label>Saldo: </label>
                            <input type="number" v-model="inputUser.balance">
                        </div>
                    </div>
                    <button class="primary-btn" @click="saveUser()">GUARDAR</button>
                </div>
            </Modal>
            <button class="primary-btn" @click="showModalSearch = true">Cargar datos</button>
            <Modal :visible="showModalSearch" @close="closeModalSearch()">
                <div class="modal-content">
                    <h2 class="modal-title">CARGAR SALDO</h2>
                    <div class="modal-body">
                        <div class="modal-input">
                            <label>NOMBRE:</label>
                            <input type="text" v-model="inputSearch">
                        </div>
                    </div>
                    <button class="primary-btn" @click="searchUser()">BUSCAR</button>
                </div>
            </Modal>
        </div>
    </div>
    <div class="container">
        <div class="container-numbers">
            <CardButton v-for="option in bettingOptions" :key="option.number" 
                :number="option.number" 
                :color="option.color"
                @clickBet = "handleClickBet"
            />
        </div>
        <div class="container-btns">
            <button class="btns" :style="{ backgroundColor: 'red'}" @click="handleClickEven">
                PARES
            </button>
            <button class="btns" :style="{ backgroundColor: 'black'}" @click="handleClickBlack">
            </button>
            <button class="btns" :style="{ backgroundColor: 'red'}" @click="handleClickRed">
            </button>
            <button class="btns" :style="{ backgroundColor: 'black'}" @click="handleClickOdd">
                IMPARES
            </button>
        </div>
        <div>
            <button class="primary-btn" @click="handleClickPlay()">JUGAR</button>
            <Modal :visible="showModalStartGame" @close="showModalStartGame = false">
                <div class="modal-content">
                    <div class="modal-title">EMPEZAR A JUGAR</div>
                    <div class="modal-body">
                        <div>SALDO ACTUAL: {{userGame.balance.toFixed(2)}}</div>
                        <div class="modal-input">
                            <label>APOSTAR: </label>
                            <input type="number" v-model="bet.betMoney">
                        </div>
                    </div>
                    <button class="primary-btn" @click="startGame()">EMPEZAR</button>
                </div>
            </Modal>
            <Modal :visible="showModalWinningNumber" @close="closeModalWinnerNumber">
                <div class="modal-content">
                    <h2 class="modal-title">RESULTADO DE LA RULETA</h2>
                    <h3 style="margin: 0; text-align: center;">{{ isWinner ? `GANASTE: ${prize}` : "PERDISTE" }}</h3>
                    <div class="modal-body" style="align-items: center;">
                        <div>Número: {{responseSpin.number}}</div>
                        <div>par/impar: {{ responseSpin.number % 2 === 0 ? 'par' : 'impar' }}</div>
                        <div>Color: {{responseSpin.color === 'black'? 'negro':'rojo'}}</div>                        
                    </div>
                    <button class="primary-btn" @click="saveGame()">Guardar</button>
                </div>
            </Modal>
        </div>
    </div>
    <div style="display: flex; flex-wrap: wrap; justify-content: center; align-items: center; gap: 10px; margin-top: 30px;">
        <div class="text">NUMEROS DE APUESTA: </div>
        <div v-for="value in bet.betList" :key="value.number" :style="{color: value.color, fontSize: '25px', fontWeight:'bold'}">{{ value.number }}</div>
    </div>
</template>

<script setup>
import { computed, ref } from 'vue';
import CardButton from './CardButton.vue';
import Modal from './Modal.vue';

let bettingOptions =  ref([...Array(37).keys()].map(number => ({
      number,
      color: Math.random() > 0.5 ? 'red' : 'black'
    })));
let showModal = ref(false);
let showModalSearch = ref(false);
let showModalStartGame = ref(false);
let showModalWinningNumber = ref(false);
let inputUser = ref(
    {
        name: "",
        balance: 0
    }
)
let inputSearch = ref('')
let prize = ref(0);
let userGame = ref(
    {
        name: "",
        balance: 0
    }
)
let bet = ref(
    {
        betMoney: 0,
        betList: [],
        typeBet: 0
    }
)
let responseSpin = ref(
    {
        number: 0,
        color: ""
    }
)
const saveUser = async () => {
    const dataInfo = {
        name: inputUser.value.name,
        cash: inputUser.value.balance
    }
    const response = await fetch("http://localhost:5049/api/User", {
        method: 'POST',
        body: JSON.stringify(dataInfo),
        headers: {"Content-type": "application/json"}
    });
    if(response.ok){
        const data = await response.json();
        userGame.value.name = data.name;
        userGame.value.balance = data.cash;
        showModal.value = false;
        inputUser.value.name="";
        inputUser.value.balance=0;
    }
    else if(response.status == 409){
        alert("Error: El nombre de usuario ya existe");
    }
    else{
        alert("Error: Ocurrio un error al guardar tus datos")
    }
}

const searchUser = async () => {
    const response = await fetch(`http://localhost:5049/api/User/${inputSearch.value}`);
    if(response.ok){
        const data = await response.json();
        userGame.value.name = data.name;
        userGame.value.balance = data.cash;
        showModalSearch.value = false;
        inputSearch.value="";
    }
    else{
        alert(`No se encontro ningun usuario con nombre: ${inputSearch.value}`)
    }
}

const closeModalSearch = () => {
    showModalSearch.value = false
    inputSearch.value="";
}

const handleClickPlay = () => {
    if(userGame.value.name == ""){
        alert("Para empezar a jugar debe registrarse primero o cargar sus datos")
        return;
    }
    if(bet.value.betList.length === 0){
        alert("Seleccione el tipo de apuesta en uno de los cuadros")
        return;
    }
    showModalStartGame.value = true;
}

const startGame = async () => {
    if(bet.value.betMoney<0){
        alert("La apuesta no puede ser negativo");
        return;
    }
    if(bet.value.betMoney > userGame.value.balance){
        alert("La apuesta supera el monto de tu saldo")
        return;
    }
    const response = await fetch(`http://localhost:5049/api/Roulette`);
    if(response.ok){
        const data = await response.json();
        responseSpin.value.number = data.number;
        responseSpin.value.color = data.color;
        showModalWinningNumber.value = true;
        await getPrize();
    }
    showModalStartGame.value = false;
}

const handleClickBet = (number, color) => {
    bet.value.betList =[{number, color}]
    bet.value.typeBet = 1;
}

const handleClickEven = () => {
    bet.value.betList = bettingOptions.value.filter((option)=>option.number%2===0 && option.color === "red");
    bet.value.typeBet = 2;
}

const handleClickOdd = () => {
    bet.value.betList = bettingOptions.value.filter((option)=>option.number%2!==0 && option.color === "black");
    bet.value.typeBet = 2;
}

const handleClickBlack = () => {
    bet.value.betList = bettingOptions.value.filter((option)=>option.color === "black");
    bet.value.typeBet = 3;
}

const handleClickRed = () => {
    bet.value.betList = bettingOptions.value.filter((option)=>option.color === "red");
    bet.value.typeBet = 3;
}

const isWinner = computed(()=>{
    return bet.value.betList.some(bet => bet.number === responseSpin.value.number && bet.color === responseSpin.value.color);
})

const saveGame = async () => {
    const dataInfo = {
        name: userGame.value.name,
        cash: userGame.value.balance
    }
    const response = await fetch(`http://localhost:5049/api/User/${userGame.value.name}`, {
        method: 'PUT',
        body: JSON.stringify(dataInfo),
        headers: {"Content-type": "application/json"}
    });
    if(response.ok){
        alert("Se guardo con exito tus datos");
    }
    closeModalWinnerNumber();
}

const getPrize = async () => {
    const response = await fetch(`http://localhost:5049/api/Roulette/prize`, {
        method: 'POST',
        body: JSON.stringify({
        typeBet: bet.value.typeBet, 
        betMoney: bet.value.betMoney, 
        }),
        headers: { "Content-type": "application/json" }
    });
    if(response.ok){
        const data = await response.json();
        prize.value = data.prize;
        if(isWinner.value){
            userGame.value.balance = userGame.value.balance + data.prize;
        }
        else{
            userGame.value.balance = userGame.value.balance - bet.value.betMoney;
        }
    }
}

const closeModalWinnerNumber = () => {
    bet.value.betMoney = 0;
    bet.value.betList = [];
    showModalWinningNumber.value = false;
}
</script>

<style scoped>
.container{
    
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
}
.container-numbers{
    display: flex;
    flex-wrap: wrap;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    padding-top: 20px;
    padding-bottom: 20px;
    padding-left: 30px;
    padding-right: 30px;
    width: 50%;
}
.container-btns{
    display: flex;
    flex-direction: row;
    justify-content: center;
}
.btns{
    color: white;
    width: 100px;
    height: 50px;
}
.primary-btn{
    padding-left: 15px;
    padding-right: 15px;
    padding-top: 5px;
    padding-bottom: 5px;
    margin: 2px;
    border: 2px solid rgb(24, 28, 131);
    border-radius: 4px;
    background-color: rgb(24, 28, 131);
    color: white;
}
.text{
    color: white; font-size: large; font-weight: bold;
}
.modal-content{
    display: flex; justify-content: center; flex-direction: column; gap: 12px;
}
.modal-title{
    text-align: center;
    font-size: larger;
    font-weight: bold;
}
.modal-body{
    font-size: medium;
    display: flex;
    flex-direction: column;
    justify-content: center;
    gap: 10px;
}
.modal-input{
    display: flex;
    flex-direction: row;
    gap: 10px;
    justify-content: space-between;
}
</style>