<script lang="ts" context="module">
    export type Move = {
        i: number;
        j: number;
        capture: boolean;
    };
    export type CellBg =
        | "plain"
        | "move"
        | "capture"
        | "prev"
        | "moved"
        | "check";
    export function normalizeState(state: CellType[][], clearPrev: boolean) {
        // const plains: Pos[] = [];
        // console.log("normalizing = ", state);
        const prevs: Pos[] = [];
        const moved: Pos[] = [];
        state.forEach((row, i) => {
            row.forEach((cell, j) => {
                if (cell.prevPos.i != i || cell.prevPos.j != j) {
                    prevs.push({
                        i: cell.prevPos.i,
                        j: cell.prevPos.j,
                    });
                    moved.push({
                        i,
                        j,
                    });
                } else {
                    if (cell.cellBg == "prev" || cell.cellBg == "moved") {
                        if (clearPrev) {
                            cell.cellBg = "plain";
                        }
                    } else {
                        if (cell.cellBg != "check") {
                            cell.cellBg = "plain";
                        }
                    }
                }
            });
        });
        console.log(prevs);
        prevs.forEach((p) => {
            state[p.i][p.j].cellBg = "prev";
        });
        moved.forEach((m) => {
            state[m.i][m.j].cellBg = "moved";
        });
        // state = state;
        return state;
    }
    export function findMovesNoSanitize(
        state: CellType[][],
        i: number,
        j: number,
        kingMoved: KingMoved,
        passantAble: Passantable,
        rookMoved: RookMoved
    ) {
        // console.log("find moves no sanitize");
        console.log("find no sanitie i j ", i, j);
        const cell = state[i][j];

        const moves: Move[] = [];

        if (cell.value == "" || cell.color == "") {
            return moves;
        }
        if (cell.value == "P") {
            let iInc = cell.color == "white" ? -1 : +1;
            let goStraight = false;
            let row = state[i + iInc];
            if (row) {
                if (row[j].value == "") {
                    moves.push({
                        i: i + iInc,
                        j,
                        capture: false,
                    });
                    goStraight = true;
                }
            }
            if (goStraight && i == (cell.color == "black" ? 1 : 6)) {
                if (state[i + iInc * 2][j].value == "") {
                    moves.push({
                        i: i + iInc * 2,
                        j,
                        capture: false,
                    });
                }
            }
            const cellI = cell.color == "black" ? 4 : 3;
            if (i == cellI) {
                const cellLeft = state[cellI][j - 1];
                const cellRight = state[cellI][j + 1];
                const tArr = [];

                if (cellLeft) {
                    tArr.push(-1);
                }
                if (cellRight) {
                    tArr.push(1);
                }
                tArr.forEach((t) => {
                    const opColor = not(cell.color);
                    if (opColor != "") {
                        const passant = passantAble[opColor];
                        if (passant) {
                            if (passant.i == cellI && passant.j == j + t) {
                                moves.push({
                                    i: cellI == 3 ? 2 : 5,
                                    j: passant.j,
                                    capture: true,
                                });
                            }
                        }
                    }
                });
            }

            [1, -1].forEach((a) => {
                let row = state[i + iInc];
                if (row) {
                    if (row[j + a] && row[j + a].color == not(cell.color)) {
                        moves.push({
                            i: i + iInc,
                            j: j + a,
                            capture: true,
                        });
                    }
                }
            });
        }
        if (cell.value == "R") {
            let rowMoves: Move[] = [];
            let colMoves: Move[] = [];
            for (let loopJ = j - 1; loopJ >= 0; loopJ--) {
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
            for (let loopI = i - 1; loopI >= 0; loopI--) {
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
            }
            for (let loopI = i + 1; loopI < 8; loopI++) {
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
                ////////console.log("here1");
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
                ////////console.log("here2");
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
                // ////////console.log("behind");
                ////////console.log("here3");
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
                ////////console.log("here4");
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
            if (!kingMoved[cell.color]) {
                const KI = cell.color == "black" ? 0 : 7;
                // console.log("Cell color = ", cell.color);
                let KingCell = state[KI][4];
                // console.log("moves bf castling = ", moves);
                if (i == KI && j == 4 && !kingMoved[cell.color]) {
                    const RQJ = 0;
                    const RKJ = 7;

                    let RookQCell = state[KI][RQJ];
                    let RookKCell = state[KI][RKJ];
                    let QueenCastle = true;
                    let kingCastle = true;
                    if (
                        RookQCell.color == cell.color &&
                        RookQCell.value == "R" &&
                        !rookMoved[cell.color]["Q"]
                    ) {
                        for (let loopJ = 3; loopJ >= 1; loopJ--) {
                            if (state[KI][loopJ].value !== "") {
                                QueenCastle = false;
                                break;
                            }
                        }
                    } else {
                        QueenCastle = false;
                    }
                    if (
                        RookKCell.color == cell.color &&
                        RookKCell.value == "R" &&
                        !rookMoved[cell.color]["K"]
                    ) {
                        for (let loopJ = 5; loopJ <= 6; loopJ++) {
                            if (state[KI][loopJ].value !== "") {
                                kingCastle = false;
                                break;
                            }
                        }
                    } else {
                        kingCastle = false;
                    }

                    // console.log("QC,KC", QueenCastle, kingCastle);
                    if (QueenCastle) {
                        moves.push({
                            i: KI,
                            j: 2,
                            capture: false,
                        });
                    }
                    if (kingCastle) {
                        moves.push({
                            i: KI,
                            j: 6,
                            capture: false,
                        });
                    }
                }
            }
        }
        return moves;
    }
    export function not(cellColor: "white" | "black" | "") {
        if (cellColor == "white") {
            return "black";
        } else if (cellColor == "black") {
            return "white";
        }
        return cellColor;
    }
    export function sanitizeMoves(
        state: CellType[][],
        moves: Move[],
        curI: number,
        curJ: number,
        kingPos: {
            black: Pos;
            white: Pos;
        },
        passantAble: Passantable
        // cell: CellType
    ) {
        let retMoves: Move[] = [];
        let curCell = state[curI][curJ];
        //@ts-ignore
        const kp = kingPos[curCell.color];
        const KI = curCell.color == "black" ? 0 : 7;
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
                prevPos: {
                    i: curI,
                    j: curJ,
                },
            };

            if (curCell.color !== "") {
                if (curCell.value == "K") {
                    let tKps = { ...kingPos };
                    tKps[curCell.color] = {
                        i: m.i,
                        j: m.j,
                    };

                    if (
                        !checkCheck(tempState, curCell.color, tKps, passantAble)
                    ) {
                        retMoves.push({ ...m });
                    }
                } else {
                    if (
                        !checkCheck(
                            tempState,
                            curCell.color,
                            kingPos,
                            passantAble
                        )
                    ) {
                        retMoves.push({ ...m });
                    }
                }
            }
        });
        if (curI == KI && curJ == 4 && curCell.value == "K") {
            console.log("king moves = ", moves);
            console.log("retMOves = ", retMoves);
            let QCF = false;
            let KCF = false;
            let tempState: CellType[][] = [];
            state.forEach((r) => {
                tempState.push([...r]);
            });
            if (kp.i == KI && kp.j == 4) {
                for (let i = 0; i < retMoves.length; i++) {
                    if (retMoves[i].i == KI && retMoves[i].j == 3) {
                        QCF = true;
                    }
                    if (retMoves[i].i == KI && retMoves[i].j == 5) {
                        KCF = true;
                    }
                }
            }
            let cloneRetMoves: Move[] = [];

            retMoves.forEach((m) => {
                if (m.i == KI && m.j == 2) {
                    if (QCF) {
                        cloneRetMoves.push({ ...m });
                    }
                } else if (m.i == KI && m.j == 6) {
                    if (KCF) {
                        cloneRetMoves.push({ ...m });
                    }
                } else {
                    cloneRetMoves.push({ ...m });
                }
            });

            retMoves = [...cloneRetMoves];
        }
        //////console.log("retmoves = ", retMoves);
        return retMoves;
    }
    export function findPossibleMoves(
        state: CellType[][],
        i: number,
        j: number,
        kingPos: {
            black: Pos;
            white: Pos;
        },
        kingMoved: KingMoved,
        rookMoved: RookMoved,
        passantAble: Passantable
        // cell: CellType
    ) {
        const moves = findMovesNoSanitize(
            state,
            i,
            j,
            kingMoved,
            passantAble,
            rookMoved
        );
        console.log("moves state = ", state);
        console.log("moves no sanitize = ", moves);
        return sanitizeMoves(state, moves, i, j, kingPos, passantAble);
        // return sanitizeMoves(state, moves, i, j, kingPos, passantAble, cell);
    }
    export function checkCheck(
        state: CellType[][],
        victimColor: "black" | "white",
        kingPos: {
            black: Pos;
            white: Pos;
        },
        passantAble: Passantable
    ) {
        const { i, j } = kingPos[victimColor];
        let check = false;
        //TODO make a system with piece count
        for (let loopI = 0; loopI < 8; loopI++) {
            for (let loopJ = 0; loopJ < 8; loopJ++) {
                if (state[loopI][loopJ].color == not(victimColor)) {
                    ////////console.log("op color");
                    const moves = findMovesNoSanitize(
                        state,
                        loopI,
                        loopJ,
                        {
                            black: true,
                            white: true,
                        },
                        passantAble,
                        {
                            black: {
                                K: true,
                                Q: true,
                            },
                            white: {
                                K: true,
                                Q: true,
                            },
                        }
                    );
                    check =
                        moves.filter((m) => {
                            return m.i == i && m.j == j && m.capture == true;
                        }).length !== 0;
                    if (check) {
                        return true;
                    }
                }
            }
        }
        return false;
    }
    export function checkmateCheck(
        state: CellType[][],
        victimColor: Turn,
        kingPos: {
            black: Pos;
            white: Pos;
        },
        passantAble: Passantable
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
                            kingPos,
                            {
                                black: true,
                                white: true,
                            },

                            {
                                black: {
                                    K: true,
                                    Q: true,
                                },
                                white: {
                                    K: true,
                                    Q: true,
                                },
                            },
                            passantAble
                            // cell
                        );
                        if (possibleMoves.length > 0) {
                            return false;
                        }
                    }
                }
            }
        }
        return true;
    }
    export function move(
        state: CellType[][],
        selectedCell: Pos,
        i: number,
        j: number,
        KingMoved: KingMoved,
        RookMoved: RookMoved,
        passantAble: Passantable,
        send: boolean,
        kingPos: KingPos,
        turn: Turn,
        roomCode: string,
        ws: WebSocket,
        promotion: boolean,
        promotePos: Pos
        // cell: CellType
    ) {
        const sCell = state[selectedCell.i][selectedCell.j];
        let vColor = not(sCell.color);
        let moves = findPossibleMoves(
            state,
            selectedCell.i,
            selectedCell.j,
            kingPos,
            KingMoved,
            RookMoved,
            passantAble
            // cell
        );
        let moveSuccess = false;
        const dCell = state[selectedCell.i][selectedCell.j];
        const KI = sCell.color == "black" ? 0 : 7;
        if (
            selectedCell.i == KI &&
            selectedCell.j == 4 &&
            i == KI &&
            (j == 2 || j == 6)
        ) {
            moves.forEach((m) => {
                if (m.i == KI) {
                    if (j == 2) {
                        state[KI][2] = {
                            value: "K",
                            cellBg: "plain",
                            color: sCell.color,
                            prevPos: {
                                i: KI,
                                j: 4,
                            },
                        };
                        state[i][3] = {
                            value: "R",
                            cellBg: "plain",
                            color: sCell.color,
                            prevPos: {
                                i: KI,
                                j: 0,
                            },
                        };
                        state[i][0] = {
                            value: "",
                            cellBg: "plain",
                            color: "",
                            prevPos: {
                                i: i,
                                j: 0,
                            },
                        };
                        state[KI][4] = {
                            value: "",
                            cellBg: "plain",
                            color: "",
                            prevPos: {
                                i: KI,
                                j: 4,
                            },
                        };
                        //@ts-ignore
                        KingMoved[sCell.color] = true;
                        moveSuccess = true;
                        //@ts-ignore
                        RookMoved[sCell.color]["Q"] = true;

                        // break;
                    } else if (j == 6) {
                        state[KI][6] = {
                            value: "K",
                            cellBg: "plain",
                            color: sCell.color,
                            prevPos: {
                                i: KI,
                                j: 4,
                            },
                        };
                        state[i][5] = {
                            value: "R",
                            cellBg: "plain",
                            color: sCell.color,
                            prevPos: {
                                i: KI,
                                j: 7,
                            },
                        };
                        state[i][7] = {
                            value: "",
                            cellBg: "plain",
                            color: "",
                            prevPos: {
                                i: i,
                                j: 7,
                            },
                        };
                        state[KI][4] = {
                            value: "",
                            cellBg: "plain",
                            color: "",
                            prevPos: {
                                i: KI,
                                j: 4,
                            },
                        };
                        moveSuccess = true;
                        //@ts-ignore
                        KingMoved[sCell.color] = true;
                        //@ts-ignore
                        RookMoved[sCell.color]["Q"] = true;

                        // break;
                    }
                } else if (m.i == i && m.j == j) {
                    state[i][j] = {
                        ...state[selectedCell.i][selectedCell.j],
                        prevPos: {
                            i: selectedCell.i,
                            j: selectedCell.j,
                        },
                    };
                    state[selectedCell.i][selectedCell.j] = {
                        cellBg: "plain",
                        color: "",
                        value: "",
                        prevPos: {
                            i: selectedCell.i,
                            j: selectedCell.j,
                        },
                    };
                    moveSuccess = true;
                    //@ts-ignore
                    KingMoved[sCell.color] = true;

                    // break;
                }
            });
        } else {
            //if not king in castle position
            console.log(moves);
            moves.forEach((m) => {
                if (m.i == i && m.j == j) {
                    state[i][j] = {
                        ...state[selectedCell.i][selectedCell.j],
                        prevPos: {
                            i: selectedCell.i,
                            j: selectedCell.j,
                        },
                    };
                    state[selectedCell.i][selectedCell.j] = {
                        cellBg: "plain",
                        color: "",
                        value: "",
                        prevPos: {
                            i: selectedCell.i,
                            j: selectedCell.j,
                        },
                    };
                    if (sCell.value == "P") {
                        //@ts-ignore
                        const passant = passantAble[vColor];
                        const promoteI = sCell.color == "black" ? 7 : 0;
                        console.log(promoteI);
                        if (i == promoteI) {
                            promotion = true;
                            promotePos = {
                                i: i,
                                j: j,
                            };
                        }
                        if (passant) {
                            if (passant.i == selectedCell.i && passant.j == j) {
                                state[passant.i][passant.j] = {
                                    cellBg: "plain",
                                    color: "",
                                    value: "",
                                    prevPos: {
                                        i: passant.i,
                                        j: passant.j,
                                    },
                                };
                            }
                        }
                        if (Math.abs(i - selectedCell.i) == 2) {
                            //@ts-ignore
                            passantAble[sCell.color] = {
                                i,
                                j,
                            };
                        }
                    }
                    if (sCell.color != "") {
                        if (sCell.value == "K") {
                            KingMoved[sCell.color] = true;
                        }
                        if (sCell.value == "R" && selectedCell.j == 0) {
                            RookMoved[sCell.color]["Q"] = true;
                        }
                        if (sCell.value == "R" && selectedCell.j == 7) {
                            RookMoved[sCell.color]["K"] = true;
                        }
                    }
                    moveSuccess = true;
                }
            });
        }
        console.log(moveSuccess);
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
            if (vColor == "black" || vColor == "white") {
                let cc = checkCheck(state, vColor, kingPos, passantAble);
                console.log("check = ", cc);
                if (cc) {
                    const { i, j } = kingPos[vColor];
                    console.log(i, j);
                    state[i][j].cellBg = "check";
                    console.log(state[i][j].cellBg);
                    if (
                        checkmateCheck(
                            state,
                            //@ts-ignore
                            vColor,
                            kingPos,
                            passantAble
                        )
                    ) {
                        alert("Checkmate");
                    }
                }
            }
            //@ts-ignore
            passantAble[vColor] = {
                black: undefined,
                white: undefined,
            };
            let move = {
                room_code: roomCode,
                i: selectedCell.i,
                j: selectedCell.j,
                k: i,
                l: j,
            };
            if (send && !promotion) {
                ws.send(
                    JSON.stringify({
                        event: Event.Move,
                        msg: JSON.stringify(move),
                    })
                );
            }

            selectedCell = {
                i: -1,
                j: -1,
            };
            // console.log("state before normalize = ",state);
            state = normalizeState(state, true);
        } else {
            if (turn == state[i][j].color) {
                console.log("ignore 2");
                state = normalizeState(state, false);
                let moves = findPossibleMoves(
                    state,
                    i,
                    j,
                    kingPos,
                    KingMoved,
                    RookMoved,
                    passantAble
                    // cell
                );
                moves.forEach((m) => {
                    state[m.i][m.j].cellBg = m.capture ? "capture" : "move";
                });
                selectedCell = {
                    i,
                    j,
                };
            }
        }
        console.log(state);
        return {
            state,
            selectedCell,
            kingPos,
            KingMoved,
            RookMoved,
            turn,
            passantAble,
            promotePos,
            promotion,
        };
    }
</script>

<script lang="ts">
    import "../../app.css";
    import type {
        CellType,
        KingMoved,
        KingPos,
        Passantable,
        Pos,
        RookMoved,
        Turn,
    } from "./+page.svelte";
    import type { WsMsg } from "./RoomsInit.svelte";
    import { Event } from "./RoomsInit.svelte";
    // export let immutState: CellType[][];
    export let ws: WebSocket;
    export let cell: CellType;
    export let i: number, j: number;
    export let state: CellType[][];
    export let turn: Turn;
    export let KingMoved: KingMoved;
    export let RookMoved: RookMoved;
    export let passantAble: Passantable;
    export let promotion: boolean;
    export let promotePos: Pos;
    export let kingPos: {
        black: Pos;
        white: Pos;
    };
    export let rotate: boolean;
    export let SELF: string;
    export let roomCode: string;
    export let selectedCell: {
        i: number;
        j: number;
    };

    let slideX = 0;
    let slideY = 0;

    $: {
        slideX = state[i][j].prevPos.j - j;
        slideY = state[i][j].prevPos.i - i;
        state[i][j].prevPos = {
            i,
            j,
        };
    }
    $: {
        if (slideX != 0 || slideY != 0) {
            setTimeout(() => {
                slideX = 0;
                slideY = 0;
            }, 0);
        }
    }

    $: cellBg = state[i][j].cellBg;
    const bgColor =
        i % 2 == 0 ? (j % 2 == 0 ? "w" : "b") : j % 2 == 0 ? "b" : "w";
    let cellClick = () => {
        if (turn == SELF) {
            console.log("selected check", selectedCell);
            if (selectedCell.i > -1 && selectedCell.j > -1) {
                if (!promotion) {
                    //@ts-ignore
                    console.log("Moving");
                    console.log("moving i j", i, j);
                    let x = move(
                        state,
                        selectedCell,
                        i,
                        j,
                        KingMoved,
                        RookMoved,
                        passantAble,
                        true,
                        kingPos,
                        turn,
                        roomCode,
                        ws,
                        promotion,
                        promotePos
                        // cell
                    );
                    selectedCell = x.selectedCell;
                    state = x.state;
                    RookMoved = x.RookMoved;
                    kingPos = x.kingPos;
                    KingMoved = x.KingMoved;
                    turn = x.turn;
                    promotePos = x.promotePos;
                    promotion = x.promotion;
                    passantAble = x.passantAble;
                    if (promotion) {
                        let msg: WsMsg<{
                            room_code: string;
                            i: number;
                            j: number;
                            promote_to: string;
                        }> = {
                            event: Event.Promote,
                            msg: {
                                room_code: roomCode,
                                i: i,
                                j: j,
                                promote_to: "Q",
                            },
                        };
                        // ws.send()
                    }
                }
            } else {
                if (turn == state[i][j].color) {
                    // console.log("normalizing");
                    console.log("ignore");
                    state = normalizeState(state, false);
                    let moves = findPossibleMoves(
                        state,
                        i,
                        j,
                        kingPos,
                        KingMoved,
                        RookMoved,
                        passantAble
                    );
                    moves.forEach((m) => {
                        state[m.i][m.j].cellBg = m.capture ? "capture" : "move";
                    });
                    selectedCell = {
                        i,
                        j,
                    };
                }
            }
        }
    };
</script>

<button
    on:click={cellClick}
    class={`relative ${bgColor == "b" ? "bg-slate-700" : "bg-lime-300/50"} ${
        rotate ? "scale-y-[-1]" : ""
    }  text-[#ff79c6] font-bold `}
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
    <div
        class={`absolute w-full h-full bg-yellow-600 opacity-40 top-0 left-0  ${
            cell.cellBg == "check" ? "block" : "hidden"
        }`}
    />
    <div
        class={`absolute w-full h-full  opacity-40 top-0 left-0  ${
            cellBg == "prev" ? "flex justify-center items-center" : "hidden"
        }`}
    >
        <div class="w-full h-full scale-50 bg-blue-700 rounded-full" />
    </div>
    <div
        class={`absolute w-full h-full  opacity-40 top-0 left-0  ${
            cellBg == "moved" ? "flex justify-center items-center" : "hidden"
        }`}
    >
        <div class="relative w-full h-full flex justify-center">
            <div class="absolute w-[20%] h-full bg-blue-700 rotate-45" />
            <div class="absolute w-[20%] h-full bg-blue-700 -rotate-45" />
        </div>
    </div>
    {#if cell.color || cell.value}
        <div
            style={`transform: translate(${(slideX * 25) / 2}vh, ${
                (slideY * 25) / 2
            }vh)`}
            class={`flex justify-center items-center duration-200`}
        >
            <img
                alt={`${cell.color}|${cell.value}`}
                src={`/images/${
                    cell.color
                }_${cell.value.toLocaleLowerCase()}.png`}
            />
        </div>
    {/if}
</button>
