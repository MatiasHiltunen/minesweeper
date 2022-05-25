<script setup>
import { computed } from '@vue/reactivity';
import { ref, onMounted, onUnmounted, watch } from 'vue'

const x = ref(10)
const y = ref(10)
const readyField = ref(null)
const mineCount = ref(0)
const gameDone = ref(false)

const useCounter = () => {

    const seconds = ref(0)
    const stop = ref(false)
    let key

    const update = () => {

        seconds.value++
    }

    const reset = () => {
        stop.value = false
        seconds.value = 0
        clearInterval(key)
        key = setInterval(update, 1000)
    }

    watch(stop, () => {
        if (stop.value) {
            clearInterval(key)
        }
    })

    onUnmounted(() => {
        clearInterval(key)
    })

    return { seconds, reset, stop }
}

const { seconds, stop, reset } = useCounter()

const setField = () => {
    reset()
    let countMines = 0
    const field = new Array(x.value).fill([]).map(a => new Array(y.value).fill('').map((a) => {
        const cellValue = Math.random() > 0.8 ? 'x' : '0'

        if (cellValue === 'x') countMines++

        return cellValue
    }))

    mineCount.value = countMines

    const checkCellsAround = (i, j) => {
        let count = 0

        for (let a = -1; a <= 1; a++) {
            for (let b = -1; b <= 1; b++) {
                if (field[i + a] && field[i + a][j + b] && field[i + a][j + b] === 'x') {
                    count++
                }
            }
        }

        return count
    };

    readyField.value = field
        .map((a, i) => {
            return a.map((b, j) => {
                if (b !== 'x') {
                    let count = checkCellsAround(i, j)
                    b = count.toString();
                }
                return [b, 0];
            })

        })
}

const allFound = computed(() => {
    if (!readyField.value) return false
    return readyField.value.every(a => a.every(b => (b[0] !== 'x' && (b[1] === 1 || b[1] === 2) || b[0] === 'x' && (b[1] === 0 || b[1] === 2))))
})

watch(allFound, ()=>{   
    console.log(allFound.value) 
    if(allFound.value) stop.value = true
})

const setCellsAround = (i, j) => {
    for (let a = -1; a <= 1; a++) {
        for (let b = -1; b <= 1; b++) {
            if (readyField.value[i + a] && readyField.value[i + a][j + b]) {

                if (readyField.value[i + a][j + b][1] === 0) {
                    readyField.value[i + a][j + b][1] = 1


                    if (readyField.value[i + a][j + b][0] == '0') {
                        setCellsAround(i + a, j + b)
                    }
                }
            }
        }
    }
}

const gameOver = () => {
    stop.value = true
    gameDone.value = true
    readyField.value.map((a, i) => a.map((b, j) => setTimeout(() => b[1] = 1, (i * 100) + (j * 100))))
}

const check = (i, j) => {

    if (readyField.value[i][j][0] == 'x') {
        gameOver()
    }

    if (readyField.value[i][j][0] == '0') {
        setCellsAround(i, j)
    } else {

        readyField.value[i][j][1] = 1
    }

}


const flag = (i, j) => {

    if (readyField.value[i][j][1] == 2) {
        readyField.value[i][j][1] = 0
    } else if (readyField.value[i][j][1] === 0) {
        readyField.value[i][j][1] = 2
    }
}

const newGame = ()=>{
    setField() 
    gameDone.value = false
}


onMounted(setField)



</script>

<template>
    <div class="main-menu" v-if="gameDone || allFound">

        <template v-if="allFound">
            <h2>All Mines Cleared!</h2>
            Time: {{ seconds }} seconds
            <button class="btn" @click="newGame">
                New Game
            </button>
        </template>

        <template v-if="gameDone">

            <h2>
                Game Over
            </h2>
            Time: {{ seconds }} seconds
            <button class="btn" @click="newGame">
                Start Again
            </button>
        </template>

    </div>
    <h3> Mines: {{ mineCount }}</h3>
    <h3>

        Time: {{ seconds }}
    </h3>
    <div v-if="readyField" class="container">
        <template v-for="(row, i) in readyField">
            <div class="row">


                <template v-for="(col, j) in row">
                    <div @click="check(i, j)" @contextmenu.prevent="flag(i, j)" v-if="col[1] == 0"
                        class="col unchecked cell">

                    </div>
                    <div v-else-if="col[1] === 2" @contextmenu.prevent="flag(i, j)" class="col flag cell">
                        <img src="https://media2.giphy.com/media/fYNSI87f937Is2sNEF/giphy.gif?cid=6c09b9529b8fc982985f617b9cba60a088006194f0d6e2f4&rid=giphy.gif&ct=s"
                            width="30" alt="">
                    </div>
                    <div v-else-if="col[0] === 'x'" class="col mine cell">

                        <img src="https://cdn.pixabay.com/photo/2013/07/12/14/10/explosion-147909__340.png" width="30"
                            alt="">
                    </div>
                    <div v-else class="col number cell">
                        {{ col[0] === '0' ? '' : col[0] }}
                    </div>
                </template>
            </div>

        </template>
    </div>

</template>

<style>
.main-menu {
    width: 100%;
    height: 100vh;
    position: fixed;
    background-color: rgba(208, 208, 208, 0.772);
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    bottom: 0px;
}

.btn {
    width: 100px;
    border: none;
    background-color: grey;
    color: aliceblue;
    border-radius: 5px;
    padding: 4px;
    margin-top: 10px;
}

.cell {
    margin: 3px;
    border-radius: 5px;
}

.unchecked {
    background-color: lightgray;
}

.number {
    background-color: white;
}

.flag {
    background-color: rgb(62, 79, 62);
    color: brown;
}

.mine {
    background-color: rgb(100, 65, 65);
}

.container {
    width: 500px;
    height: 500px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    background-color: grey;
    padding: 5px;
    border-radius: 5px;
}

.row {
    display: flex;
    justify-content: space-between;
    height: 100%;
    width: 100%
}

.col {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%
}
</style>