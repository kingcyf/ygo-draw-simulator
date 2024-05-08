<template>
    <div class="wrapper">
        <div class="simulator">
            <div class="deck panel">
                <div class="toolbar panel-title">
                    卡组共有<input type="text" v-model="numOfCards">张卡
                </div>
                <div class="list">
                    <div v-for="card in deck"
                         class="card"
                         :class="{invalid: !card.name || !(card.num > 0)}"
                    >
                        <button @click="removeCard(card)" title="删除">×</button>
                        <input class="name" type="text" v-model="card.name" placeholder="卡片名称">
                        <span>×</span>
                        <input class="num" type="text" v-model="card.num" placeholder="数量">
                    </div>
                    <div class="card others" v-if="numOfOthers">
                        <span class="name">其它卡片</span>
                        <span>×</span>
                        <span class="num">{{numOfOthers}}</span>
                    </div>
                </div>
                <div class="footer">
                    <input class="name" type="text" placeholder="卡片名称" v-model="cardName">
                    <span>×</span>
                    <input class="num" type="text" v-model="cardNum" placeholder="数量">
                    <button @click="addCard" :disabled="!cardName || !(cardNum > 0)" title="添加">+</button>
                </div>
            </div>

            <div class="main">
                <condition-builder v-model="condition"
                                   root
                                   :card-names="cardNames"
                                   style="flex: 1 1; min-height: 0;"/>

                <div class="panel sim">
                    <div class="panel-title">
                        第一回合抽 <input type="text" v-model="firstTurnDrawCards"> 张卡
                        <div class="fill"></div>
                        模拟洗牌 <input type="text" v-model="simulationTimes"> 次
                        <button style="margin-left: .5em" @click="simulate">计算！</button>
                    </div>
                    <div class="content">
                        <table>
                            <tr>
                                <th v-for="(val, index) in result">回合 {{index + 1}}</th>
                            </tr>
                            <tr>
                                <td v-for="val in result">{{val | percent}}</td>
                            </tr>
                        </table>
                    </div>
                </div>
            </div>
        </div>
        <div class="footer">Author: x6ud</div>
    </div>
</template>

<script>
    import ConditionBuilder from './ConditionBuilder.vue'

    function randomInt(min, max) {
        min = Math.ceil(min);
        max = Math.floor(max);
        return Math.floor(Math.random() * (max - min + 1)) + min;
    }

    function shuffle(arr) {
        for (let i = 0, last = arr.length - 1; i < last; ++i) {
            const j = randomInt(i, last);
            [arr[i], arr[j]] = [arr[j], arr[i]];
        }
        return arr;
    }

    function createVerificationFunction(condition) {
        switch (condition.type) {
            case 'and': {
                const children = condition.children.map(createVerificationFunction);
                return function (deck, draws) {
                    for (let i = 0, len = children.length; i < len; ++i) {
                        if (!children[i](deck, draws)) {
                            return false;
                        }
                    }
                    return true;
                };
            }
            case 'or': {
                const children = condition.children.map(createVerificationFunction);
                return function (deck, draws) {
                    for (let i = 0, len = children.length; i < len; ++i) {
                        if (children[i](deck, draws)) {
                            return true;
                        }
                    }
                    return false;
                };
            }
            case 'single': {
                const cards = condition.cards.map(card => card.name);
                const symbol = condition.symbol;
                const num = Number(condition.num);
                return function (deck, draws) {
                    let count = 0;
                    for (let i = 0; i < draws; ++i) {
                        const cardName = deck[i];
                        for (let j = 0, len = cards.length; j < len; ++j) {
                            if (cardName === cards[j]) {
                                count += 1;
                                break;
                            }
                        }
                    }
                    switch (symbol) {
                        case 'gt':
                            return count > num;
                        case 'eq':
                            return count === num;
                        case 'lt':
                            return count < num;
                        case 'neq':
                            return count !== num;
                    }
                    return false;
                };
            }
        }
    }

    export default {
        components: {
            ConditionBuilder
        },
        data() {
            return {
                numOfCards: 40,
                deck: [],
                cardName: '',
                cardNum: 3,
                condition: {
                    type: 'and',
                    children: []
                },
                firstTurnDrawCards: 5,
                simulationTimes: 50000,
                result: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
            };
        },
        computed: {
            numOfOthers() {
                return Math.max(0, Number(this.numOfCards) - this.deck.reduce((sum, card) => (sum += Math.max(0, Number(card.num) || 0)), 0));
            },
            cardNames() {
                const map = {};
                this.deck.forEach(card => {
                    map[card.name] = card;
                });
                return Object.keys(map);
            }
        },
        filters: {
            percent(val) {
                return (val * 100).toFixed(1) + '%';
            }
        },
        methods: {
            addCard() {
                const existed = this.deck.find(card => card.name === this.cardName);
                if (existed) {
                    existed.num += Number(this.cardNum);
                } else {
                    this.deck.push({name: this.cardName, num: Number(this.cardNum)});
                }
                this.cardName = '';
            },
            removeCard(card) {
                this.deck = this.deck.filter(curr => curr !== card);
            },
            createDeck() {
                const ret = [];
                this.deck.forEach(card => {
                    for (let i = 0, len = Number(card.num); i < len; ++i) {
                        ret.push(card.name);
                    }
                });
                for (let i = 0, len = this.numOfOthers; i < len; ++i) {
                    ret.push('');
                }
                return ret;
            },
            simulate() {
                const deck = this.createDeck();
                const verify = createVerificationFunction(this.condition);
                const firstTurnDrawCards = Number(this.firstTurnDrawCards);
                const simulationTimes = Number(this.simulationTimes);
                const result = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0];
                for (let i = 0; i < simulationTimes; ++i) {
                    shuffle(deck);
                    for (let j = 0, len = result.length; j < len; ++j) {
                        const draws = j + firstTurnDrawCards;
                        if (verify(deck, draws)) {
                            result[j] += 1;
                        }
                    }
                }
                for (let i = 0, len = result.length; i < len; ++i) {
                    result[i] /= simulationTimes;
                }
                this.result = result;
            }
        }
    };
</script>

<style lang="scss" scoped>
    .wrapper {
        display: flex;
        flex-direction: column;
        width: 100%;
        height: 100%;

        & > .footer {
            padding: 0 8px 4px 8px;
            color: #999;
            font-size: 12px;
            text-align: right;
        }

        & > .simulator {
            display: flex;
            width: 100%;
            flex: 1 1;
            min-height: 0;
            box-sizing: border-box;
            padding: 8px 8px 4px 8px;

            button {
                outline: none;
                user-select: none;
            }

            .panel {
                box-sizing: border-box;
                border: solid 1px #aaa;
                overflow: hidden;

                .panel-title {
                    padding: 8px;
                    background: #f2f2f2;
                    border-bottom: solid 1px #aaa;
                }
            }

            .deck {
                display: flex;
                flex-direction: column;
                width: 260px;
                height: 100%;

                .toolbar {
                    display: flex;
                    align-items: center;
                    justify-content: flex-end;

                    input {
                        width: 4em;
                        text-align: center;
                        margin: 0 .5em;
                        height: 28px;
                        box-sizing: border-box;
                    }
                }

                .footer {
                    display: flex;
                    align-items: center;
                    padding: 8px;
                    background: #f2f2f2;
                    border-top: solid 1px #aaa;

                    input {
                        height: 28px;
                        box-sizing: border-box;

                        &.name {
                            flex: 1 1;
                            min-width: 0;
                        }

                        &.num {
                            width: 2em;
                            text-align: center;
                        }
                    }

                    span {
                        margin: 0 .5em;
                    }

                    button {
                        width: 28px;
                        height: 28px;
                        margin-left: 8px;
                    }
                }

                .list {
                    flex: 1 1;
                    min-height: 0;
                    overflow: auto;

                    .card {
                        display: flex;
                        align-items: center;
                        height: 32px;
                        border-bottom: solid 1px #eee;

                        &.invalid {
                            background: #fde2e2;
                        }

                        .name {
                            flex: 1 1;
                            min-width: 0;
                            padding: 0 4px;
                        }

                        .num {
                            width: 2em;
                            text-align: center;
                        }

                        input {
                            height: 100%;
                            border: none;
                            background: none;
                        }

                        button {
                            height: 100%;
                            border: none;
                            cursor: pointer;
                        }
                    }
                }
            }

            .main {
                display: flex;
                flex-direction: column;
                flex: 1 1;
                min-width: 0;
                margin-left: 8px;

                & > *:not(:last-child) {
                    margin-bottom: 8px;
                }

                .sim {
                    .panel-title {
                        display: flex;
                        align-items: center;

                        input {
                            width: 6em;
                            height: 28px;
                            text-align: center;
                            padding: 0 4px;
                            box-sizing: border-box;
                            margin: 0 .5em;
                        }

                        button {
                            height: 28px;
                            box-sizing: border-box;
                        }

                        .fill {
                            flex: 1 1;
                        }
                    }

                    .content {
                        padding: 8px;

                        table {
                            border-collapse: collapse;

                            th, td {
                                border: solid 1px #aaa;
                                text-align: center;
                                font-weight: normal;
                                padding: 4px 8px;
                            }
                        }
                    }
                }
            }
        }
    }
</style>