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
    let promotion = false;
    let promotePos = {
        i: -1,
        j: -1,
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

<div
    class={`${
        promotion ? "flex" : "hidden"
    } absolute w-screen h-screen  justify-center items-center z-30`}
>
    <div class={`flex w-fit h-fit  flex-col px-4 py-2 bg-slate-600 rounded-md`}>
        <div class="text-4xl w-full text-center">PROMOTE TO:</div>
        <div class="flex flex-row gap-4">
            {#each ["q", "r", "b", "h"] as V}
                <button
                    on:click={() => {
                        if (promotePos.i > -1 && promotePos.j > -1) {
                            const color = promotePos.i == 7 ? "black" : "white";
                            state[promotePos.i][promotePos.j] = {
                                value: V.toUpperCase(),
                                cellBg: "plain",
                                color,
                            };
                        }
                        promotion = false;
                        promotePos = {
                            i: -1,
                            j: -1,
                        };
                    }}
                >
                    <img
                        alt={`${V}|${"black"}`}
                        src={`/images/${(promotePos.i == 7
                            ? "black"
                            : "white"
                        ).toLocaleLowerCase()}_${V}.png`}
                    />
                </button>
            {/each}
        </div>
    </div>
</div>
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
                    bind:promotion
                    bind:promotePos
                />
            {/each}
        </div>
    {/each}
</div>
