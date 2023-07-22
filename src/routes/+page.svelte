<script lang="ts" context="module">
    export type CellType = {
        value: string;
        color: "black" | "white" | "";
        cellBg: CellBg;
    };
    export type Pos = {
        i: number;
        j: number;
    };
    export type Turn = "white" | "black";
    export type RookMoved = {
        white: {
            Q: boolean;
            K: boolean;
        };
        black: {
            Q: boolean;
            K: boolean;
        };
    };
    export type KingMoved = {
        white: boolean;
        black: boolean;
    };
    export type Passantable = {
        white: Pos | undefined;
        black: Pos | undefined;
    };
</script>

<script lang="ts">
    import Cell, { type CellBg } from "./Cell.svelte";
    import "../app.css";
    let turn: Turn = "white";
    let RookMoved = {
        white: {
            Q: false,
            K: false,
        },
        black: {
            Q: false,
            K: false,
        },
    };
    let KingMoved = {
        white: false,
        black: false,
    };
    let passantAble: Passantable = {
        black: undefined,
        white: undefined,
    };
    // let prev: CellType[][];
    let state: CellType[][] = [
        [
            { color: "black", value: "R", cellBg: "plain" },
            { color: "black", value: "H", cellBg: "plain" },
            { color: "black", value: "B", cellBg: "plain" },

            { color: "black", value: "Q", cellBg: "plain" },
            { color: "black", value: "K", cellBg: "plain" },
            { color: "black", value: "B", cellBg: "plain" },
            { color: "black", value: "H", cellBg: "plain" },
            { color: "black", value: "R", cellBg: "plain" },
        ],
        [
            { color: "black", value: "P", cellBg: "plain" },
            { color: "black", value: "P", cellBg: "plain" },
            { color: "black", value: "P", cellBg: "plain" },
            { color: "black", value: "P", cellBg: "plain" },
            { color: "black", value: "P", cellBg: "plain" },
            { color: "black", value: "P", cellBg: "plain" },
            { color: "black", value: "P", cellBg: "plain" },
            { color: "black", value: "P", cellBg: "plain" },
        ],
        [
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
        ],
        [
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
        ],
        [
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
        ],
        [
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
            { color: "", value: "", cellBg: "plain" },
        ],
        [
            { color: "white", value: "P", cellBg: "plain" },
            { color: "white", value: "P", cellBg: "plain" },
            { color: "white", value: "P", cellBg: "plain" },
            { color: "white", value: "P", cellBg: "plain" },
            { color: "white", value: "P", cellBg: "plain" },
            { color: "white", value: "P", cellBg: "plain" },
            { color: "white", value: "P", cellBg: "plain" },
            { color: "white", value: "P", cellBg: "plain" },
        ],
        [
            { color: "white", value: "R", cellBg: "plain" },
            { color: "white", value: "H", cellBg: "plain" },
            { color: "white", value: "B", cellBg: "plain" },
            { color: "white", value: "Q", cellBg: "plain" },
            { color: "white", value: "K", cellBg: "plain" },
            { color: "white", value: "B", cellBg: "plain" },
            { color: "white", value: "H", cellBg: "plain" },
            { color: "white", value: "R", cellBg: "plain" },
        ],
    ];
    let kingPos = {
        black: {
            i: 0,
            j: 4,
        },
        white: {
            i: 7,
            j: 4,
        },
    };
    let selectedCell: {
        i: number;
        j: number;
    } = {
        i: -1,
        j: -1,
    };
</script>

<div class="grid grid-rows-[repeat(8,1fr)] w-[100vh] h-[100vh]">
    {#each state as row, i}
        <div class="grid grid-cols-[repeat(8,1fr)]">
            {#each row as cell, j}
                <Cell
                    bind:cell
                    {i}
                    {j}
                    bind:state
                    bind:selectedCell
                    bind:kingPos
                    bind:turn
                    bind:KingMoved
                    bind:RookMoved
                    bind:passantAble
                />
            {/each}
        </div>
    {/each}
</div>
