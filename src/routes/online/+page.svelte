<script lang="ts" context="module">
    export type CellType = {
        value: string;
        color: "black" | "white" | "";
        cellBg: CellBg;
        prevPos: Pos;
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
    export type Animate = {
        from: Pos;
        to: Pos;
    };
    export type KingPos = {
        black: Pos;
        white: Pos;
    };
</script>

<script lang="ts">
    import Cell, { type CellBg } from "./Cell.svelte";
    import "../../app.css";
    import RoomsInit, { Event, Status } from "./RoomsInit.svelte";

    // let ws = new WebSocket(
    //     `${
    //         location.protocol == "https:" ? "wss" : "ws"
    //     }://${location.hostname}${location.port ? "/" + location.port : ""}/ws`
    // );

    // let ws = new WebSocket("wss://rustws.cleverapps.io/ws");
    let ws = new WebSocket("ws://localhost:8080/ws");
    let roomCode = "";
    let joinRoomCode = "";
    let RoomGenerateStatus: Status = Status.None;
    let RoomJoinStatus: Status = Status.None;
    let OppStatus: Status = Status.None;
    let turn: Turn = "white";
    let SELF: Turn = "white";
    let selfId = "";
    let name = "";
    let captured: CellType[] = [];
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
    let rotate = false;
    let state: CellType[][] = [
        [
            {
                color: "black",
                value: "R",
                cellBg: "plain",
                prevPos: { i: 0, j: 0 },
            },
            {
                color: "black",
                value: "H",
                cellBg: "plain",
                prevPos: { i: 0, j: 1 },
            },
            {
                color: "black",
                value: "B",
                cellBg: "plain",
                prevPos: { i: 0, j: 2 },
            },
            {
                color: "black",
                value: "Q",
                cellBg: "plain",
                prevPos: { i: 0, j: 3 },
            },
            {
                color: "black",
                value: "K",
                cellBg: "plain",
                prevPos: { i: 0, j: 4 },
            },
            {
                color: "black",
                value: "B",
                cellBg: "plain",
                prevPos: { i: 0, j: 5 },
            },
            {
                color: "black",
                value: "H",
                cellBg: "plain",
                prevPos: { i: 0, j: 6 },
            },
            {
                color: "black",
                value: "R",
                cellBg: "plain",
                prevPos: { i: 0, j: 7 },
            },
        ],
        [
            {
                color: "black",
                value: "P",
                cellBg: "plain",
                prevPos: { i: 1, j: 0 },
            },
            {
                color: "black",
                value: "P",
                cellBg: "plain",
                prevPos: { i: 1, j: 1 },
            },
            {
                color: "black",
                value: "P",
                cellBg: "plain",
                prevPos: { i: 1, j: 2 },
            },
            {
                color: "black",
                value: "P",
                cellBg: "plain",
                prevPos: { i: 1, j: 3 },
            },
            {
                color: "black",
                value: "P",
                cellBg: "plain",
                prevPos: { i: 1, j: 4 },
            },
            {
                color: "black",
                value: "P",
                cellBg: "plain",
                prevPos: { i: 1, j: 5 },
            },
            {
                color: "black",
                value: "P",
                cellBg: "plain",
                prevPos: { i: 1, j: 6 },
            },
            {
                color: "black",
                value: "P",
                cellBg: "plain",
                prevPos: { i: 1, j: 7 },
            },
        ],
        [
            { color: "", value: "", cellBg: "plain", prevPos: { i: 2, j: 0 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 2, j: 1 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 2, j: 2 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 2, j: 3 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 2, j: 4 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 2, j: 5 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 2, j: 6 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 2, j: 7 } },
        ],
        [
            { color: "", value: "", cellBg: "plain", prevPos: { i: 3, j: 0 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 3, j: 1 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 3, j: 2 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 3, j: 3 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 3, j: 4 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 3, j: 5 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 3, j: 6 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 3, j: 7 } },
        ],
        [
            { color: "", value: "", cellBg: "plain", prevPos: { i: 4, j: 0 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 4, j: 1 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 4, j: 2 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 4, j: 3 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 4, j: 4 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 4, j: 5 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 4, j: 6 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 4, j: 7 } },
        ],
        [
            { color: "", value: "", cellBg: "plain", prevPos: { i: 5, j: 0 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 5, j: 1 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 5, j: 2 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 5, j: 3 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 5, j: 4 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 5, j: 5 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 5, j: 6 } },
            { color: "", value: "", cellBg: "plain", prevPos: { i: 5, j: 7 } },
        ],
        [
            {
                color: "white",
                value: "P",
                cellBg: "plain",
                prevPos: { i: 6, j: 0 },
            },
            {
                color: "white",
                value: "P",
                cellBg: "plain",
                prevPos: { i: 6, j: 1 },
            },
            {
                color: "white",
                value: "P",
                cellBg: "plain",
                prevPos: { i: 6, j: 2 },
            },
            {
                color: "white",
                value: "P",
                cellBg: "plain",
                prevPos: { i: 6, j: 3 },
            },
            {
                color: "white",
                value: "P",
                cellBg: "plain",
                prevPos: { i: 6, j: 4 },
            },
            {
                color: "white",
                value: "P",
                cellBg: "plain",
                prevPos: { i: 6, j: 5 },
            },
            {
                color: "white",
                value: "P",
                cellBg: "plain",
                prevPos: { i: 6, j: 6 },
            },
            {
                color: "white",
                value: "P",
                cellBg: "plain",
                prevPos: { i: 6, j: 7 },
            },
        ],
        [
            {
                color: "white",
                value: "R",
                cellBg: "plain",
                prevPos: { i: 7, j: 0 },
            },
            {
                color: "white",
                value: "H",
                cellBg: "plain",
                prevPos: { i: 7, j: 1 },
            },
            {
                color: "white",
                value: "B",
                cellBg: "plain",
                prevPos: { i: 7, j: 2 },
            },
            {
                color: "white",
                value: "Q",
                cellBg: "plain",
                prevPos: { i: 7, j: 3 },
            },
            {
                color: "white",
                value: "K",
                cellBg: "plain",
                prevPos: { i: 7, j: 4 },
            },
            {
                color: "white",
                value: "B",
                cellBg: "plain",
                prevPos: { i: 7, j: 5 },
            },
            {
                color: "white",
                value: "H",
                cellBg: "plain",
                prevPos: { i: 7, j: 6 },
            },
            {
                color: "white",
                value: "R",
                cellBg: "plain",
                prevPos: { i: 7, j: 7 },
            },
        ],
    ];
    $: whiteCaptured = captured.filter((c) => c.color == "white");
    $: blackCaptured = captured.filter((c) => c.color == "black");
    let kingPos: KingPos = {
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

<div class="relative">
    <RoomsInit
        bind:RoomJoinStatus
        bind:RoomGenerateStatus
        bind:OppStatus
        bind:roomCode
        bind:joinRoomCode
        bind:ws
        bind:SELF
        bind:selfId
        bind:state
        bind:turn
        bind:kingMoved={KingMoved}
        bind:rookMoved={RookMoved}
        bind:passantAble
        bind:kingPos
        bind:promotion
        bind:promotePos
        bind:selectedCell
        bind:rotate
        bind:name
        bind:captured
    />
    <div
        class={`${
            promotion ? "flex" : "hidden"
        } absolute w-screen h-screen  justify-center items-center z-10`}
    >
        <div
            class={`flex w-fit h-fit  flex-col px-4 py-2 bg-slate-600 rounded-md`}
        >
            <div class="text-4xl w-full text-center">PROMOTE TO:</div>
            <div class="flex flex-row gap-4">
                {#each ["q", "r", "b", "h"] as V}
                    <button
                        on:click={() => {
                            if (promotePos.i > -1 && promotePos.j > -1) {
                                const color =
                                    promotePos.i == 7 ? "black" : "white";
                                state[promotePos.i][promotePos.j] = {
                                    value: V.toUpperCase(),
                                    cellBg: "plain",
                                    color,
                                    prevPos: {
                                        i: promotePos.i,
                                        j: promotePos.j,
                                    },
                                };
                                let msg = {
                                    event: Event.Promote,
                                    msg: JSON.stringify({
                                        room_code: roomCode,
                                        i: promotePos.i,
                                        j: promotePos.j,
                                        promote_to: V.toUpperCase(),
                                    }),
                                };
                                ws.send(JSON.stringify(msg));
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
    <div class="grid grid-cols-[100vh_auto] gap-4 w-screen h-screen bg-purple-500">
        <div
            class={`grid grid-rows-[repeat(8,1fr)] w-[100vh] h-[100vh] ${
                rotate ? "scale-y-[-1]" : ""
            }`}
        >
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
                            bind:ws
                            bind:roomCode
                            bind:SELF
                            bind:rotate
                            bind:captured
                        />
                    {/each}
                </div>
            {/each}
        </div>
        <div class="flex flex-col justify-between py-4 w-full h-full">
            <div class="flex h-16 w-full bg-green-300 flex-row gap-2">
                {#each whiteCaptured as c}
                    <div>
                        <img
                            alt={`${c.color}|${c.value}`}
                            src={`/images/${
                                c.color
                            }_${c.value.toLocaleLowerCase()}.png`}
                        />
                    </div>
                {/each}
            </div>
            <div
                class={`text-4xl ${turn == "white" ? "text-white" : "text-black"}`}
            >
                Turn : {turn}
            </div>
            <div class="flex h-16 w-full bg-blue-300 flex-row gap-2">
                {#each blackCaptured as c}
                    <div>
                        <img
                            alt={`${c.color}|${c.value}`}
                            src={`/images/${
                                c.color
                            }_${c.value.toLocaleLowerCase()}.png`}
                        />
                    </div>
                {/each}
            </div>
        </div>
    </div>
</div>
