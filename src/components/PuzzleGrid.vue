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
        <div class="score-board">
            <h2>Score: {{ score }}</h2>
        </div>
    </div>
</template>

<script>
export default {
    name: 'PuzzleGrid',
    data() {
        return {
            staticCells: [{ value: 1, x: 3, y: 7 }, { value: 5, x: 3, y: 6 }],
            fallingCell: null,
            nextCell: { value: 9, x: 3, y: 0 }, // 初期の落ちるセルの設定
            intervalId: null,
            score: 0
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
                backgroundColor: 'rgb(196, 200, 243)',
                border: '1px solid lightgray',
                zIndex: 1,
                display: 'flex',
                justifyContent: 'center',
                alignItems: 'center',
                fontSize: '16px', // 必要に応じてフォントサイズを調整
                color: 'black' // 必要に応じて文字の色を調整
            };
        },
        updateScore(numberOfCellsRemoved) {
            this.score += numberOfCellsRemoved; // 削除したセルの数だけスコアを増加
            this.$emit('update-score', this.score); // スコアの変更を親コンポーネントに通知
        },
        //隣接チェック
        checkAdjacent(cells) {
            console.log('*********', cells)
            const adjacent = (cell1, cell2) => {
                console.log(cell1, cell2);
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
                        console.log('@@@@@@@@@@@', subset)
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
        moveCellsDownInColumn(x, minY) {
            // 指定された列のセルを移動させる
            this.staticCells.forEach(cell => {
                if (cell.x === x && cell.y < minY) {
                    cell.y += 1;
                }
            });
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
                console.log('*************', cellsToRemove);
                this.staticCells = this.staticCells.filter(cell => !cellsToRemove.has(cell));

                // スコアを更新
                this.updateScore(cellsToRemove.size);

                // セルを削除した後、列内のセルを下に移動
                const columnsToUpdate = [...new Set(Array.from(cellsToRemove).map(cell => cell.x))];
                columnsToUpdate.forEach(x => this.moveCellsDownInColumn(x, Math.max(...Array.from(cellsToRemove).map(cell => cell.y))));
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
        },
        moveLeft() {
            if (this.fallingCell) {
                const newCell = { ...this.fallingCell, x: this.fallingCell.x - 1 };
                if (!this.isCollision(newCell)) {
                    this.fallingCell.x -= 1;
                }
            }
        },
        moveRight() {
            if (this.fallingCell) {
                const newCell = { ...this.fallingCell, x: this.fallingCell.x + 1 };
                if (!this.isCollision(newCell)) {
                    this.fallingCell.x += 1;
                }
            }
        },
        moveDown() {
            if (this.fallingCell) {
                const newCell = { ...this.fallingCell, y: this.fallingCell.y + 1 };
                if (this.isCollision(newCell)) {
                    this.staticCells.push(this.fallingCell);
                    this.checkForCompleteLines();
                    this.fallingCell = null;
                } else {
                    this.fallingCell.y += 1;
                }
            }
        },
        handleKeyDown(event) {
            switch (event.key) {
                case 'ArrowLeft':
                    this.moveLeft();
                    break;
                case 'ArrowRight':
                    this.moveRight();
                    break;
                case 'ArrowDown':
                    this.moveDown();
                    break;
                default:
                    break;
            }
        }
    },
    created() {
        this.intervalId = setInterval(this.moveCellsDown, 1000);
        window.addEventListener('keydown', this.handleKeyDown);
    },
    beforeUnmount() {
        clearInterval(this.intervalId);
        window.removeEventListener('keydown', this.handleKeyDown);
    }
}
</script>

<style scoped>
.grid-container {
    display: flex;
    justify-content: center;
    align-items: center;
    /* 半透明な白色の背景 */
    background-color: rgba(255, 255, 255, 0.373);
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

.score-board h2 {
    font-size: 24px;
    color: #333;
    margin-left: 100px;
}
</style>