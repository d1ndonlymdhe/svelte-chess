<script lang="ts" context="module">
    export enum Event {
        Move = "Move",
        GameOver = "GameOver",
        Start = "Start",
        GetCode = "GetCode",
        ConnectWith = "ConnectWith",
        OppReady = "OppReady",
    }

    export enum Status {
        Success,
        Loading,
        None,
    }

    export type WsMsg<T> = {
        event: Event;
        msg: T;
    };
</script>

<script lang="ts">
    import type {
        CellType,
        KingMoved,
        Passantable,
        RookMoved,
        Turn,
    } from "../offline/+page.svelte";
    export let RoomJoinStatus: Status;
    export let RoomGenerateStatus: Status;
    export let OppStatus: Status;
    export let roomCode: string;
    export let joinRoomCode: string;
    export let ws: WebSocket;
    export let SELF: Turn;
    export let selfId: String = "";
    export let state: CellType[][];
    export let turn: Turn;
    export let kingMoved: KingMoved;
    export let rookMoved: RookMoved;
    export let passantAble: Passantable;
    export let kingPos: KingPos;
    export let promotion: boolean;
    export let promotePos: Pos;
    export let selectedCell: Pos;
    export let rotate: boolean;
    import "../../app.css";

    import { move, normalizeState } from "./Cell.svelte";
    import type { KingPos, Pos } from "./+page.svelte";
    ws.onopen = () => {
        ws.onmessage = (res) => {
            const data = JSON.parse(res.data) as WsMsg<any>;
            const event = data.event;
            switch (event) {
                case Event.Move:
                    const { i, j, k, l } = data.msg as {
                        i: number;
                        j: number;
                        k: number;
                        l: number;
                    };
                    console.log("moving");

                    turn = SELF;
                    console.log(data.msg);
                    let x = move(
                        state,
                        { i, j },
                        k,
                        l,
                        kingMoved,
                        rookMoved,
                        passantAble,
                        false,
                        kingPos,
                        turn,
                        roomCode,
                        ws,
                        promotion,
                        promotePos
                    );
                    selectedCell = x.selectedCell;
                    state = x.state;
                    rookMoved = x.RookMoved;
                    kingPos = x.kingPos;
                    kingMoved = x.KingMoved;
                    // turn = x.turn;
                    promotePos = x.promotePos;
                    promotion = x.promotion;
                    passantAble = x.passantAble;
                    normalizeState(state, false);
                    break;
                case Event.GameOver:
                    console.log("Game over");
                    break;
                case Event.Start:
                    console.log(data.msg);
                    selfId = data.msg;
                    console.log("start");
                    break;
                case Event.GetCode:
                    console.log(data.msg);
                    const msg = data.msg as {
                        id: string;
                        code: string;
                    };
                    console.log(typeof data.msg);
                    roomCode = msg.code;
                    RoomGenerateStatus = Status.Success;
                    console.log("ready");
                    break;
                case Event.ConnectWith:
                    console.log("connect with");
                    console.log(data.msg);
                    if (roomCode == "") {
                        roomCode = joinRoomCode;
                    }
                    wsSend<String>({
                        event: Event.OppReady,
                        msg: roomCode,
                    });
                    break;
                case Event.OppReady:
                    console.log("Opponent Ready");
                    OppStatus = Status.Success;
                    break;
                default:
                    console.log(data);
                    alert("Invalid Event");
            }
        };
    };
    function generateRoom() {
        if (
            ws.OPEN &&
            RoomGenerateStatus == Status.None &&
            RoomJoinStatus == Status.None
        ) {
            let generateMessage: WsMsg<string> = {
                event: Event.GetCode,
                msg: "",
            };
            SELF = "white";
            rotate = false;
            RoomGenerateStatus = Status.Loading;
            wsSend(generateMessage);
        }
    }
    function joinRoom() {
        if (
            ws.OPEN &&
            RoomGenerateStatus == Status.None &&
            RoomJoinStatus == Status.None
        ) {
            if (joinRoomCode.length != 0) {
                let joinMsg: WsMsg<string> = {
                    event: Event.ConnectWith,
                    msg: joinRoomCode,
                };
                SELF = "black";
                rotate = true;
                wsSend(joinMsg);
            }
        }
    }
    function wsSend<T>(msg: WsMsg<T>) {
        ws.send(JSON.stringify(msg));
    }
</script>

<div
    class={`${
        OppStatus == Status.Success ? "hidden" : "flex"
    }  justify-center items-center text-2xl w-screen h-screen text-black absolute top-0 left-0 backdrop-blur-lg z-20`}
>
    <div class="flex flex-col gap-4 bg-lime-300 px-8 py-2 rounded-md">
        <div id="roomCode" />
        <p class="text-center">
            {#if RoomGenerateStatus == Status.Loading}
                Loading
            {:else if RoomGenerateStatus == Status.Success}
                {roomCode}
            {/if}
        </p>
        <button class="bg-red-400 py-2 rounded-md" on:click={generateRoom}
            >Generate Room</button
        >
        <label>
            <p class="text-center">Enter Room Code</p>
            <input
                class="bg-red-400 py-2 rounded-md px-2"
                bind:value={joinRoomCode}
            />
        </label>
        <button class="bg-red-400 py-2 rounded-md" on:click={joinRoom}
            >Join Room</button
        >
    </div>
</div>
