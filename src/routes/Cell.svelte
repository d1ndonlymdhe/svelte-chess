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
    import type { CellType, Pos, Turn } from "./+page.svelte";
    export let cell: CellType;
    export let i: number, j: number;
    export let state: CellType[][];
    export let turn: Turn;
    // export let passantee : Pos[];
    // export let passenter : Pos[];
    export let kingPos: {
        black: Pos;
        white: Pos;
    };
    export let selectedCell: {
        i: number;
        j: number;
    };
    // export let readyToMove:boolean;

    $: cellBg = state[i][j].cellBg;
    const bgColor =
        i % 2 == 0 ? (j % 2 == 0 ? "w" : "b") : j % 2 == 0 ? "b" : "w";

    function sanitizeMoves(
        state: CellType[][],
        moves: Move[],
        curI: number,
        curJ: number,
        kingPos: {
            black: Pos;
            white: Pos;
        }
    ) {
        let retMoves: Move[] = [];
        let curCell = state[curI][curJ];
        ////console.log("moves to be sanitied = ", moves);
        // alert("cur cell="+curCell.value);
        console.log("checking moves")
        moves.forEach((m) => {
            let tempState: CellType[][] = [];
            state.forEach((r) => {
                tempState.push([...r]);
            });
            tempState[m.i][m.j] = { ...curCell };
            tempState[curI][curJ] = {
                cellBg: "plain",
                color: "",
                value: "",
            };
            // tempState[0][0] = {
            //     cellBg:"capture",
            //     color:"white",
            //     value:"HEllo"
            // }
            console.log("check check start")
            if (curCell.color !== "") {
                // alert("king selected")
                console.log("checking ",curCell)
                if (curCell.value == "K") {
                    let tKps = { ...kingPos };
                    tKps[curCell.color] = {
                        i: m.i,
                        j: m.j,
                    };
                    if (!checkCheck(tempState, curCell.color, tKps)) {
                        retMoves.push({ ...m });
                    }
                } else {
                    if (!checkCheck(tempState, curCell.color, kingPos)) {
                        retMoves.push({ ...m });
                    }
                }
            }
            console.log("check check end")

        });
        console.log("end checking moves")

        ////console.log("retmoves = ", retMoves);
        return retMoves;
    }
    function findMovesNoSanitize(
        state: CellType[][],
        i: number,
        j: number,
        kingPos: {
            black: Pos;
            white: Pos;
        }
    ) {
        const cell = state[i][j];
        // //////console.log("cell = ",cell)
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
            // //////console.log(cell.color == "black" ? 1 : 6);
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
                // //////console.log("here1");
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
                // //////console.log("here2");
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
            for (let loopI = i - 1; loopI >= 0; loopI--) {
                // //////console.log("behind");
                // //////console.log("here3");
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
                // //////console.log("here4");
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
                    // //////console.log(i + loopI, j + loopJ);
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
                    // //////console.log(i + loopI, j + loopJ);
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
        if (cell.value == "Q") {
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
                            loopI += iInc;
                            loopJ += jInc;
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
            let rowMoves: Move[] = [];
            let colMoves: Move[] = [];
            // for(let loopJ  = j;loopJ<8;loopJ++)
            for (let loopJ = j - 1; loopJ >= 0; loopJ--) {
                //////console.log("here1");
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
                //////console.log("here2");
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
            for (let loopI = i - 1; loopI >= 0; loopI--) {
                // //////console.log("behind");
                //////console.log("here3");
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
                //////console.log("here4");
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
                            loopI += iInc;
                            loopJ += jInc;
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
        if (cell.value == "K") {
            for (let iINc = -1; iINc <= 1; iINc++) {
                const dRow = state[i + iINc];
                if (dRow) {
                    for (let jInc = -1; jInc <= 1; jInc++) {
                        const dCell = dRow[j + jInc];
                        if (dCell && dCell.color !== cell.color) {
                            moves.push({
                                i: i + iINc,
                                j: j + jInc,
                                capture: dCell.color == not(cell.color),
                            });
                        }
                    }
                }
            }
        }
        // //////console.log(moves);
        return moves;
        // return sanitizeMoves(state, moves, i, j, kingPos);
    }
    function findPossibleMoves(
        state: CellType[][],
        i: number,
        j: number,
        kingPos: {
            black: Pos;
            white: Pos;
        }
    ) {
        const cell = state[i][j];
        // //////console.log("cell = ",cell)
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
            // //////console.log(cell.color == "black" ? 1 : 6);
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
                // //////console.log("here1");
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
                // //////console.log("here2");
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
            for (let loopI = i - 1; loopI >= 0; loopI--) {
                // //////console.log("behind");
                // //////console.log("here3");
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
                // //////console.log("here4");
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
                    // //////console.log(i + loopI, j + loopJ);
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
                    // //////console.log(i + loopI, j + loopJ);
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
        if (cell.value == "Q") {
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
                            loopI += iInc;
                            loopJ += jInc;
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
            let rowMoves: Move[] = [];
            let colMoves: Move[] = [];
            // for(let loopJ  = j;loopJ<8;loopJ++)
            for (let loopJ = j - 1; loopJ >= 0; loopJ--) {
                //////console.log("here1");
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
                //////console.log("here2");
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
                // //////console.log("behind");
                //////console.log("here3");
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
                //////console.log("here4");
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
                            loopI += iInc;
                            loopJ += jInc;
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
        if (cell.value == "K") {
            for (let iINc = -1; iINc <= 1; iINc++) {
                const dRow = state[i + iINc];
                if (dRow) {
                    for (let jInc = -1; jInc <= 1; jInc++) {
                        const dCell = dRow[j + jInc];
                        if (dCell && dCell.color !== cell.color) {
                            moves.push({
                                i: i + iINc,
                                j: j + jInc,
                                capture: dCell.color == not(cell.color),
                            });
                        }
                    }
                }
            }
        }
        // //////console.log(moves);
        // return moves;
        return sanitizeMoves(state, moves, i, j, kingPos);
    }
    function checkCheck(
        state: CellType[][],
        victimColor: "black" | "white",
        kingPos: {
            black: Pos;
            white: Pos;
        }
    ) {
        const { i, j } = kingPos[victimColor];
        let check = false;
        // const enemyPos: Pos[] = [];
        //TODO make a system with piece count
        //console.log("victim color = ",victimColor )
        console.log("Check start");

        for (let loopI = 0; loopI < 8; loopI++) {
            for (let loopJ = 0; loopJ < 8; loopJ++) {
                if (state[loopI][loopJ].color == not(victimColor)) {
                    //////console.log("op color");
                    const moves = findMovesNoSanitize(
                        state,
                        loopI,
                        loopJ,
                        kingPos
                    );
                    console.log("i,j = ", i, j);
                    console.log("moves = ", moves);
                    check =
                        moves.filter((m) => {
                            return m.i == i && m.j == j && m.capture == true;
                        }).length !== 0;
                    if (check) {
                        console.log("cehck end");
                        return true;
                    }
                }
            }
        }
        console.log("cehck end");
        return false;
    }
    function checkmateCheck(
        state: CellType[][],
        victimColor: Turn,
        kingPos: {
            black: Pos;
            white: Pos;
        }
    ) {
        for (let i = 0; i < 8; i++) {
            let row = state[i];
            if (row) {
                for (let j = 0; j < 8; j++) {
                    let cell = row[j];
                    if (cell.color == victimColor) {
                        let possibleMoves = findPossibleMoves(
                            state,
                            i,
                            j,
                            kingPos
                        );
                        if (possibleMoves.length > 0) {
                            console.log("move from = ", i, j);
                            console.log("cm moves = ", possibleMoves);
                            return false;
                        }
                    }
                }
            }
        }

        return true;
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
        state = state;
    }
</script>

<button
    on:click={() => {
        if (selectedCell.i > -1 && selectedCell.j > -1) {
            //@ts-ignore
            let vColor = not(state[selectedCell.i][selectedCell.j].color);
            //console.log("color = ", vColor);
            let moves = findPossibleMoves(
                state,
                selectedCell.i,
                selectedCell.j,
                kingPos
            );
            let moveSuccess = false;
            const dCell = state[selectedCell.i][selectedCell.j];
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
            if (moveSuccess) {
                let x = not(turn);
                if (x != "") {
                    //@ts-ignore
                    turn = x;
                }
                if (dCell.value == "K") {
                    if (dCell.color != "") {
                        kingPos[dCell.color] = {
                            i,
                            j,
                        };
                    }
                }

                if (
                    checkmateCheck(
                        state,
                        //@ts-ignore
                        vColor,
                        kingPos
                    )
                ) {
                    alert("Checkmate");
                }
                selectedCell = {
                    i: -1,
                    j: -1,
                };
                normalizeState(state);
            } else {
                // selectedCell = {
                //     i: -1,
                //     j: -1,
                // };
                if (turn == state[i][j].color) {
                    normalizeState(state);
                    let moves = findPossibleMoves(state, i, j, kingPos);
                    moves.forEach((m) => {
                        state[m.i][m.j].cellBg = m.capture ? "capture" : "move";
                    });
                    selectedCell = {
                        i,
                        j,
                    };
                }
            }
        } else {
            if (turn == state[i][j].color) {
                normalizeState(state);
                let moves = findPossibleMoves(state, i, j, kingPos);
                moves.forEach((m) => {
                    state[m.i][m.j].cellBg = m.capture ? "capture" : "move";
                });
                selectedCell = {
                    i,
                    j,
                };
            }
        }
    }}
    class={`relative ${
        bgColor == "b" ? "bg-slate-700" : "bg-lime-300/50"
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
        <!-- {cell.color}|{cell.value} -->
        <div class="flex justify-center items-center">
            <img
                alt={`${cell.color}|${cell.value}`}
                src={`/images/${
                    cell.color
                }_${cell.value.toLocaleLowerCase()}.png`}
            />
        </div>
    {/if}
</button>
