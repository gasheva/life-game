<template>
    <div>
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

import {computed, onMounted, ref} from 'vue';

const CELL_SIZE_PX = 10;
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
    }
});

const emit = defineEmits(['onClick']);

const matrix = ref(
    new Array(props.heightCellsNumber).fill(
        new Array(props.widthCellsNumber)
    )
);

const canvasHeightPx = computed(() => {
    return props.heightCellsNumber * CELL_SIZE_PX;
});

const canvasWidthPx = computed(() => {
    return props.widthCellsNumber * CELL_SIZE_PX;
});

const onClick = () => {
};

const getCellByCoord = (x, y) => {
    return [Math.ceil( x/CELL_SIZE_PX), Math.ceil(y/CELL_SIZE_PX)];
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
        return 1;
    }
    return 0;
};

function draw() {
    const canvas = document.getElementById('canvas');
    if (canvas.getContext) {
        const ctx = canvas.getContext('2d');
        for (let i = 0; i < canvasHeightPx.value; i += CELL_SIZE_PX) {
            for (let j = 0; j < canvasWidthPx.value; j += CELL_SIZE_PX) {
                matrix[i][j] = getCellState(matrix.value, j, i);
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
            matrix.value[y][x] = 0;
            drawCell(ctx, j, i);
        }
    }
}

// 1 px is for border => size - 2
function drawCell(ctx, x = 0, y = 0, backgroundColour = props.deadCellsColour, borderColour = props.borderColour, width = CELL_SIZE_PX, height = CELL_SIZE_PX,) {
    ctx.fillStyle = backgroundColour;
    ctx.fillRect(x, y, width, height);
    ctx.strokeStyle = borderColour;
    ctx.lineWidth = 1;
    ctx.strokeRect(x, y, width, height);
}

onMounted(() => {
    reset();
});
</script>

<style scoped>

</style>