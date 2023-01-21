<template>
    <div>
        <button @click="reset">RESET</button>
        <button @click="play">START</button>
        <button @click="">STOP</button>
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

const matrix =
    new Array(props.heightCellsNumber).fill().map(_ =>
        new Array(props.widthCellsNumber).fill(CellStates.DEAD)
    );
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
    drawCell(ctx, topLeftX, topLeftY, props.liveCellsColour);
    const [numberX, numberY] = getCellByCoord(topLeftX, topLeftY);
    matrix[numberY][numberX] = CellStates.ALIVE;
};

const getCellLeftTopCoords = (x, y) => {
    const [numberX, numberY] = getCellByCoord(x, y);
    return [CELL_SIZE_PX * numberX, CELL_SIZE_PX * numberY];
};

const getCellByCoord = (x, y) => {
    return [Math.floor(x / CELL_SIZE_PX), Math.floor(y / CELL_SIZE_PX)];
};

const getCellState = (prevState = matrix, x, y) => {
    const getNeighboursCount = (_state, _x, _y) => {
        let res = 0;
        res += _x === 0 ? 0 : _state[_x - 1][_y]; // left
        res += _y === 0 ? 0 : _state[_x][_y - 1]; // top
        res += (_x === _state[0].length - 1) ? 0 : _state[_x + 1][_y]; // right
        res += (_y === _state.length - 1) ? 0 : _state[_x][_y + 1]; // bottom

        res += (_x === 0 || _y === 0) ? 0 : _state[_x - 1][_y - 1]; // left-top
        res += (_x === 0 || _y === _state.length - 1) ? 0 : _state[_x - 1][_y + 1]; // left-bottom
        res += (_x === _state[0].length - 1 || _y === 0) ? 0 : _state[_x + 1][_y - 1]; // right-top
        res += (_x === _state[0].length - 1 || _y === _state.length - 1) ? 0 : _state[_x + 1][_y + 1]; // right-bottom

        return res;
    };

    const neighboursCount = getNeighboursCount(prevState, x, y);
    if (neighboursCount === 2 || neighboursCount === 3) {
        return CellStates.ALIVE;
    }
    return CellStates.DEAD;
};

function draw() {
    const canvas = document.getElementById('canvas');
    if (canvas.getContext) {
        const ctx = canvas.getContext('2d');
        for (let i = 0; i < canvasHeightPx.value; i += CELL_SIZE_PX) {
            for (let j = 0; j < canvasWidthPx.value; j += CELL_SIZE_PX) {
                matrix[i][j] = getCellState(matrix, j, i);
                drawCell(ctx, j, i);
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
            // matrix.value[i][j] = 0;
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

const hasAliveCells = () => {

};

const play = () => {
    playing = true;
    while (playing) {

    }
};

</script>

<style scoped>

</style>