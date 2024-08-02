<template>
    <div class="grid-container">
        <div class="grid">
            <div v-for="n in 48" :key="n" class="grid-cell"></div>
            <div v-for="cell in staticCells" :key="cell.x + '-' + cell.y" :style="cellStyle(cell)">
                {{ cell.value }}
            </div>
            <div v-if="fallingCell" :style="cellStyle(fallingCell)">
                {{ fallingCell.value }}
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'PuzzleGrid',
    data() {
        return {
            staticCells: [{ value: 1, x: 3, y: 7 }],
            fallingCell: null,
            nextCell: { value: 9, x: 3, y: 0 }, // 初期の落ちるセルの設定
            intervalId: null
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
        //隣接チェック
        checkAdjacent(cells) {
            console.log('*********',cells)
            const adjacent = (cell1, cell2) => {
                console.log(cell1,cell2);
                return (
                    (Math.abs(cell1.x - cell2.x) === 1 && cell1.y === cell2.y) || // 横の隣接
                    (Math.abs(cell1.y - cell2.y) === 1 && cell1.x === cell2.x)    // 縦の隣接
                );
            };

            for (let i = 0; i < cells.length - 1; i++) {
                let foundAdjacent = false;
                for (let j = i + 1; j < cells.length; j++) {
                    if (adjacent(cells[i], cells[j])) {
                        foundAdjacent = true;
                        break;
                    }
                }
                if (!foundAdjacent) return false;
            }
            return true;
        },

        // 行や列内のセルの合計が10になる組み合わせを探す
        findCellsToRemoveInRow(rowCells) {
            const result = [];
            for (let i = 0; i < (1 << rowCells.length); i++) {
                const subset = [];
                let sum = 0;
                for (let j = 0; j < rowCells.length; j++) {
                    if (i & (1 << j)) {
                        console.log('@@@@@@@@@@@',subset)
                        subset.push(rowCells[j]);
                        sum += rowCells[j].value;
                    }
                }
                if (sum === 10 && this.checkAdjacent(subset)) {
                    result.push(subset);
                }
            }
            return result;
        },

        // 行や列内で合計が10になるセルを見つけて削除
        checkForCompleteLines() {
            const cellsToRemove = new Set();

            // 行ごとの合計をチェック
            for (let y = 0; y < 8; y++) {
                const rowCells = this.staticCells.filter(cell => cell.y === y);
                const rowsToRemove = this.findCellsToRemoveInRow(rowCells);
                rowsToRemove.forEach(set => set.forEach(cell => cellsToRemove.add(cell)));
            }

            // 列ごとの合計をチェック
            for (let x = 0; x < 6; x++) {
                const columnCells = this.staticCells.filter(cell => cell.x === x);
                const columnsToRemove = this.findCellsToRemoveInRow(columnCells);
                columnsToRemove.forEach(set => set.forEach(cell => cellsToRemove.add(cell)));
            }

            // 完了したセルを削除
            if (cellsToRemove.size > 0) {
                this.staticCells = this.staticCells.filter(cell => !cellsToRemove.has(cell));
            }
        },

        isCollision(newCell) {
            // セルがグリッド外に出ているか、他のセルと衝突するかチェック
            return (
                newCell.x < 0 ||
                newCell.x >= 6 ||
                newCell.y >= 8 ||
                this.staticCells.some(cell => cell.x === newCell.x && cell.y === newCell.y)
            );
        },
        moveCellsDown() {
            console.log('called move');
            if (this.fallingCell) {
                // 現在落ちているセルがある場合
                const newCell = { ...this.fallingCell, y: this.fallingCell.y + 1 };
                console.log('newCell', newCell);

                if (this.isCollision(newCell)) {
                    // 一番下に到達したか、他のセルと衝突した場合
                    this.staticCells.push(this.fallingCell);
                    this.checkForCompleteLines(); // 完了したセルをチェックして削除
                    this.fallingCell = null; // 落ちているセルがなくなった
                } else {
                    this.fallingCell.y += 1; // セルを1つ下に移動
                }
            } else {
                console.log('new cell');
                console.log('this.fallingCell-1', this.fallingCell);
                // 落ちているセルがない場合、新しいセルを生成
                this.fallingCell = { ...this.nextCell, y: 0 };
                console.log('this.fallingCell-2', this.fallingCell);

                console.log('this.nextCell', this.nextCell);
                this.nextCell = { value: Math.floor(Math.random() * 10), x: Math.floor(Math.random() * 6), y: 0 };
                console.log('this.nextCell-2', this.nextCell);
            }
        }
    },
    created() {
        this.intervalId = setInterval(this.moveCellsDown, 1000);
    },
    beforeUnmount() {
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