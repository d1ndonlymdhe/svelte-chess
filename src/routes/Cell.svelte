<script lang="ts" context="module">
    export type Move = {
        i: number;
        j: number;
        capture: boolean;
    };
    export type CellBg = "plain" | "move" | "capture";
</script>

<script lang="ts">
    import "../app.css";
    import type { CellType } from "./+page.svelte";
    export let cell: CellType;
    export let i: number, j: number;
    export let state: CellType[][];
    export let selectedCell: {
        i: number;
        j: number;
    };
    // export let readyToMove:boolean;

    $: cellBg = state[i][j].cellBg;
    const bgColor =
        i % 2 == 0 ? (j % 2 == 0 ? "w" : "b") : j % 2 == 0 ? "b" : "w";
    function findPossibleMoves(state: CellType[][], i: number, j: number) {
        const cell = state[i][j];
        const moves: Move[] = [];
        if (cell.value == "") {
            return moves;
        }
        if (cell.value == "P") {
            let iInc = cell.color == "white" ? -1 : +1;
            let goStraight = false;
            if (state[i + iInc][j].value == "") {
                moves.push({
                    i: i + iInc,
                    j,
                    capture: false,
                });
                goStraight = true;
            }
            console.log(cell.color == "black" ? 1 : 6);
            if (goStraight && i == (cell.color == "black" ? 1 : 6)) {
                if (state[i + iInc * 2][j].value == "") {
                    moves.push({
                        i: i + iInc * 2,
                        j,
                        capture: false,
                    });
                }
            }
            // if(state[])

            [1, -1].forEach((a) => {
                if (
                    state[i + iInc][j + a] &&
                    state[i + iInc][j + a].color == not(cell.color)
                ) {
                    moves.push({
                        i: i + iInc,
                        j: j + a,
                        capture: true,
                    });
                }
            });
        }
        if (cell.value == "R") {
            let rowMoves: Move[] = [];
            let colMoves: Move[] = [];
            // for(let loopJ  = j;loopJ<8;loopJ++)
            for (let loopJ = j - 1; loopJ >= 0; loopJ--) {
                console.log("here1");
                if (state[i][loopJ].color == cell.color) {
                    break;
                }
                if (state[i][loopJ].color == not(cell.color)) {
                    rowMoves.push({
                        i,
                        j: loopJ,
                        capture: true,
                    });
                    break;
                } else if (state[i][loopJ].color == "") {
                    rowMoves.push({
                        i,
                        j: loopJ,
                        capture: false,
                    });
                }
            }
            for (let loopJ = j + 1; loopJ < 8; loopJ++) {
                console.log("here2");
                if (state[i][loopJ].color == cell.color) {
                    break;
                    // rowMoves = [];
                }
                if (state[i][loopJ].color == not(cell.color)) {
                    rowMoves.push({
                        i,
                        j: loopJ,
                        capture: true,
                    });
                    break;
                } else if (state[i][loopJ].color == "") {
                    rowMoves.push({
                        i,
                        j: loopJ,
                        capture: false,
                    });
                }
            }
            for (let loopI = i - 1; loopI > 0; loopI--) {
                // console.log("behind");
                console.log("here3");
                if (state[loopI][j].color == cell.color) {
                    break;
                    // colMoves = [];
                }
                if (state[loopI][j].color == not(cell.color)) {
                    colMoves.push({
                        i: loopI,
                        j,
                        capture: true,
                    });
                    break;
                } else if (state[loopI][j].color == "") {
                    colMoves.push({
                        i: loopI,
                        j,
                        capture: false,
                    });
                }
            }
            for (let loopI = i + 1; loopI < 8; loopI++) {
                console.log("here4");
                if (state[loopI][j].color == cell.color) {
                    break;
                }
                if (state[loopI][j].color == not(cell.color)) {
                    colMoves.push({
                        i: loopI,
                        j,
                        capture: true,
                    });
                    break;
                } else if (state[loopI][j].color == "") {
                    colMoves.push({
                        i: loopI,
                        j,
                        capture: false,
                    });
                }
                // colMoves=[]
            }
            colMoves.forEach((c) => {
                moves.push(c);
            });
            rowMoves.forEach((r) => {
                moves.push(r);
            });
        }
        if (cell.value == "H") {
            [2, -2].forEach((loopI) => {
                [-1, 1].forEach((loopJ) => {
                    console.log(i + loopI, j + loopJ);
                    const di = i + loopI;
                    const dj = j + loopJ;
                    const dRow = state[di];
                    if (dRow) {
                        const dCell = dRow[dj];
                        if (dCell && dCell.color != cell.color) {
                            moves.push({
                                i: di,
                                j: dj,
                                capture: dCell.color == not(cell.color),
                            });
                        }
                    }
                });
            });
            [1, -1].forEach((loopI) => {
                [-2, 2].forEach((loopJ) => {
                    console.log(i + loopI, j + loopJ);
                    const di = i + loopI;
                    const dj = j + loopJ;
                    const dRow = state[di];
                    if (dRow) {
                        const dCell = dRow[dj];
                        if (dCell && dCell.color != cell.color) {
                            moves.push({
                                i: di,
                                j: dj,
                                capture: dCell.color == not(cell.color),
                            });
                        }
                    }
                });
            });
        }
        if (cell.value == "B") {
            [
                {
                    iInc: 1,
                    jInc: 1,
                    limi: 8,
                    limj: 8,
                },
                {
                    iInc: -1,
                    jInc: 1,
                    limi: 0,
                    limj: 8,
                },
                {
                    iInc: -1,
                    jInc: -1,
                    limi: 0,
                    limj: 0,
                },
                {
                    iInc: 1,
                    jInc: -1,
                    limi: 8,
                    limj: 0,
                },
            ].forEach((incs) => {
                const { iInc, jInc, limi, limj } = incs;
                let loopI = i + iInc;
                let loopJ = j + jInc;
                while (
                    (limi > 0 ? loopI < limi : loopI >= limi) &&
                    (limj > 0 ? loopJ < limj : loopJ >= limj)
                ) {
                    const dRow = state[loopI];
                    if (dRow) {
                        const dCell = dRow[loopJ];

                        if (dCell.value == "") {
                            moves.push({
                                i: loopI,
                                j: loopJ,
                                capture: false,
                            });
                            loopI+=iInc;
                            loopJ+=jInc;
                        } else {
                            if (dCell.color == not(cell.color)) {
                                moves.push({
                                    i: loopI,
                                    j: loopJ,
                                    capture: true,
                                });
                            }
                            break;
                        }
                    }
                }
            });
        }
        console.log(moves);
        return moves;
    }
    function not(cellColor: "white" | "black" | "") {
        if (cellColor == "white") {
            return "black";
        } else if (cellColor == "black") {
            return "white";
        }
        return cellColor;
    }
    function normalizeState(state: CellType[][]) {
        state.forEach((row) => {
            row.forEach((cell) => {
                cell.cellBg = "plain";
            });
        });
    }
    function move(
        ci: number,
        cj: number,
        di: number,
        dj: number,
        state: CellType[][],
        moves: Move[]
    ) {
        moves.forEach((m) => {
            if (m.i == di && m.j == dj) {
                state[di][dj] = { ...state[ci][cj] };
                state[ci][cj] = {
                    cellBg: "plain",
                    color: "",
                    value: "",
                };
                return true;
            }
        });
        return false;
    }
</script>

<button
    on:click={() => {
        console.log("clicked");
        if (selectedCell.i > -1 && selectedCell.j > -1) {
            console.log("moving");

            let moves = findPossibleMoves(
                state,
                selectedCell.i,
                selectedCell.j
            );
            let moveSuccess = false;

            moves.forEach((m) => {
                if (m.i == i && m.j == j) {
                    state[i][j] = { ...state[selectedCell.i][selectedCell.j] };
                    state[selectedCell.i][selectedCell.j] = {
                        cellBg: "plain",
                        color: "",
                        value: "",
                    };
                    moveSuccess = true;
                }
            });
            console.log(moveSuccess);
            // let moveSuccess = move(
            //     selectedCell.i,
            //     selectedCell.j,
            //     i,
            //     j,
            //     state,
            //     moves
            // );
            selectedCell = {
                i: -1,
                j: -1,
            };
            normalizeState(state);
        } else {
            console.log("generating moves");
            normalizeState(state);
            let moves = findPossibleMoves(state, i, j);
            moves.forEach((m) => {
                state[m.i][m.j].cellBg = m.capture ? "capture" : "move";
            });
            selectedCell = {
                i,
                j,
            };
        }
    }}
    class={`relative ${
        bgColor == "b" ? "bg-black" : "bg-lime-300/50"
    } text-[#ff79c6] font-bold`}
>
    <div
        class={`absolute w-full h-full bg-green-300 opacity-40 top-0 left-0 ${
            cellBg == "move" ? "block" : "hidden"
        }`}
    />
    <div
        class={`absolute w-full h-full bg-red-500 opacity-40 top-0 left-0  ${
            cellBg == "capture" ? "block" : "hidden"
        }`}
    />
    {#if cell.color || cell.value}
        {cell.color}|{cell.value}
    {/if}
</button>
