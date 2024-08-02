<template>
    <div class="grid-container">
        <div class="grid">
            <div v-for="n in 48" :key="n" class="grid-cell"></div>
            <div v-for="cell in cells" :key="cell.x + '-' + cell.y" :style="cellStyle(cell)">
                {{ cell.value }}
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'PuzzleGrid',
    data() {
        return {
            cells: [{ value: 1, x: 2, y: 3 }]
        }
    },
    methods: {
        cellStyle(cell) {
            return {
                position: 'absolute',
                top: `${cell.y * 55}px`, // セルのサイズ（50px）と間隔（5px）を考慮
                left: `${cell.x * 55}px`, // セルのサイズ（50px）と間隔（5px）を考慮
                width: '50px',
                height: '50px',
                backgroundColor: 'aquamarine',
                border: '1px solid lightgray',
                zIndex: 1
            };
        },
        moveCellsDown() {
            console.log('call moveCellsDown')
            //TODO:forEachが嫌だ
            this.cells.forEach(cell => {
                if (cell.y < 7) {
                    cell.y += 1;
                }
            });
        },
    },
    created() {
        console.log('created')
        this.intervalId = setInterval(this.moveCellsDown, 1000);
    },
    beforeUnmount() {
        console.log('beforeDestroy')
        clearInterval(this.intervalId);
    }
}
</script>

<style scoped>
.grid-container {
    display: flex;
    justify-content: center;
    align-items: center;
}

.grid {
    display: grid;
    grid-template-columns: repeat(6, 50px);
    grid-template-rows: repeat(8, 50px);
    gap: 5px;
    position: relative;
}

.grid-cell {
    width: 50px;
    height: 50px;
    background-color: lightgray;
    border: 1px solid lightgray;
}
</style>