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
            const difColor = state[i + iInc][j].color == not(cell.color);
            if (state[i + iInc][j].value == "" || difColor) {
                moves.push({
                    i: i + iInc,
                    j,
                    capture: difColor,
                });
            }

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
        if (selectedCell.i > -1 && selectedCell.j > -1) {
            let moves = findPossibleMoves(
                state,
                selectedCell.i,
                selectedCell.j
            );
            let moveSuccess = move(
                selectedCell.i,
                selectedCell.j,
                i,
                j,
                state,
                moves
            );
            selectedCell = {
                i:-1,
                j:-1
            }
            normalizeState(state);
        } else {
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
