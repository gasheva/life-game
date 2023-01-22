<template>
    <div>
        <button @click="reset">RESET</button>
        <button @click="play">START</button>
        <button @click="stop">STOP</button>
        <canvas
                id="canvas"
                :height="canvasHeightPx"
                :width="canvasWidthPx"
                @click="onClick"
        >

        </canvas>
    </div>
</template>

<script>
export default {
    name: 'TheGrid'
};
</script>
<script setup>

import {computed, onMounted} from 'vue';

const CELL_SIZE_PX = 30;
const props = defineProps({
    widthCellsNumber: {
        type: Number,
        default: 20,
    },
    heightCellsNumber: {
        type: Number,
        default: 20,
    },
    liveCellsColour: {
        type: String,
        default: '#cc3434'
    },
    deadCellsColour: {
        type: String,
        default: '#ffffff'
    },
    borderColour: {
        type: String,
        default: '#000000'
    },
    timeOfGenerationInMs: {
        type: Number,
        default: 1000,
    }
});

const CellStates = {
    DEAD: 0,
    ALIVE: 1,
};

const emit = defineEmits(['onClick']);

const createMatrix = () => {
    return new Array(props.heightCellsNumber).fill().map(_ =>
        new Array(props.widthCellsNumber).fill(CellStates.DEAD)
    );
};

let matrix = createMatrix();

let playing = false;


const canvasHeightPx = computed(() => {
    return props.heightCellsNumber * CELL_SIZE_PX;
});

const canvasWidthPx = computed(() => {
    return props.widthCellsNumber * CELL_SIZE_PX;
});

const onClick = (e) => {
    if (playing) return;
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');
    const [topLeftX, topLeftY] = getCellLeftTopCoords(e.offsetX, e.offsetY);
    const [numberX, numberY] = getCellByCoord(topLeftX, topLeftY);
    matrix[numberY][numberX] = matrix[numberY][numberX] === CellStates.DEAD ? CellStates.ALIVE : CellStates.DEAD;
    drawCell(ctx, topLeftX, topLeftY, matrix[numberY][numberX] === CellStates.DEAD ? props.deadCellsColour : props.liveCellsColour);
};

const getCellLeftTopCoords = (x, y) => {
    const [numberX, numberY] = getCellByCoord(x, y);
    return [CELL_SIZE_PX * numberX, CELL_SIZE_PX * numberY];
};

const getCellByCoord = (x, y) => {
    return [Math.floor(x / CELL_SIZE_PX), Math.floor(y / CELL_SIZE_PX)];
};

function draw() {
    const canvas = document.getElementById('canvas');
    if (canvas.getContext) {
        const ctx = canvas.getContext('2d');
        for (let i = 0; i < canvasHeightPx.value; i += CELL_SIZE_PX) {
            for (let j = 0; j < canvasWidthPx.value; j += CELL_SIZE_PX) {
                const [x, y] = getCellByCoord(j, i);
                drawCell(ctx, j, i, matrix[x][y] === CellStates.ALIVE ? props.liveCellsColour : props.deadCellsColour);
            }
        }

    }
}

function reset() {
    const canvas = document.getElementById('canvas');
    if (!canvas.getContext) return;
    const ctx = canvas.getContext('2d');
    for (let i = 0; i < canvasHeightPx.value; i += CELL_SIZE_PX) {
        for (let j = 0; j < canvasWidthPx.value; j += CELL_SIZE_PX) {
            const [x, y] = getCellByCoord(j, i);
            matrix[y][x] = 0;
            drawCell(ctx, j, i);
        }
    }
}

// 1 px is for border => size - 2
function drawCell(ctx, leftTopX = 0, leftTopY = 0, backgroundColour = props.deadCellsColour, borderColour = props.borderColour, width = CELL_SIZE_PX, height = CELL_SIZE_PX,) {
    ctx.fillStyle = backgroundColour;
    ctx.fillRect(leftTopX, leftTopY, width, height);
    ctx.strokeStyle = borderColour;
    ctx.lineWidth = 1;
    ctx.strokeRect(leftTopX, leftTopY, width, height);
}

onMounted(() => {
    reset();
});


/* GAME */

let aliveCellsCounter = 0;

const countAliveCells = () => {
    matrix.forEach(ar => {
        ar.forEach(cell => cell === CellStates.ALIVE && aliveCellsCounter++);
    });
};


const rules = (neighboursCount) => {
    if (neighboursCount === 2) {
        return CellStates.ALIVE;
    }
    return CellStates.DEAD;
};

const getCellNextState = (prevState = matrix, i, j, _rules = rules) => {
    const getNeighboursCount = (_state, _i, _j) => {
        let res = 0;
        res += _i === 0 ? 0 : _state[_i - 1][_j]; // left
        res += _j === 0 ? 0 : _state[_i][_j - 1]; // top
        res += (_i === _state[0].length - 1) ? 0 : _state[_i + 1][_j]; // right
        res += (_j === _state.length - 1) ? 0 : _state[_i][_j + 1]; // bottom

        res += (_i === 0 || _j === 0) ? 0 : _state[_i - 1][_j - 1]; // left-top
        res += (_i === 0 || _j === _state.length - 1) ? 0 : _state[_i - 1][_j + 1]; // left-bottom
        res += (_i === _state[0].length - 1 || _j === 0) ? 0 : _state[_i + 1][_j - 1]; // right-top
        res += (_i === _state[0].length - 1 || _j === _state.length - 1) ? 0 : _state[_i + 1][_j + 1]; // right-bottom

        return res;
    };

    const neighboursCount = getNeighboursCount(prevState, i, j);
    return _rules(neighboursCount);
};


const stop = () => {
    playing = false;
};

const play = async () => {
    countAliveCells();
    playing = true;
    while (playing && aliveCellsCounter) {
        const wait = async () => {
            let timerNumber = null;
            const timer = new Promise((resolve) => {
                timerNumber = setTimeout(() => resolve(), props.timeOfGenerationInMs);
            });
            await timer;
            timerNumber && clearTimeout(timerNumber);
        };
        await wait();

        if (playing) {
            matrix = calculateNextStep();
            draw();
        }
    }
};

const calculateNextStep = () => {
    const newMatrix = createMatrix();

    for (let i = 0; i < matrix.length; i++) {
        for (let j = 0; j < matrix[0].length; j++) {
            const cellState = getCellNextState(matrix, i, j);
            newMatrix[i][j] = cellState;

            const updateCounter = () => {
                if (matrix[i][j] !== cellState) {
                    aliveCellsCounter += (cellState.DEAD ? -1 : 1);
                }
            };

            updateCounter();
        }
    }

    return newMatrix;
};

</script>

<style scoped>

</style>