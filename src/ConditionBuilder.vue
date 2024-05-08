<template>
    <div class="condition"
         v-if="value && value.type"
         :class="value && value.type"
    >
        <template v-if="value.type === 'and' || value.type === 'or'">
            <div class="title">
                满足以下
                <select v-model="value.type">
                    <option value="and">全部</option>
                    <option value="or">任一</option>
                </select>
                条件
                <div class="fill"></div>
                <button title="删除" v-if="!root" @click="remove">×</button>
            </div>
            <div class="children">
                <template v-if="value.children">
                    <condition-builder v-for="child in value.children"
                                       :value="child"
                                       :card-names="cardNames"
                                       @remove="onRemoveChild"/>
                </template>
                <div class="buttons">
                    <a href="javascript:void(0);" @click="addGroup">添加条件组</a>
                    <a href="javascript:void(0);" @click="addSingle">添加条件</a>
                </div>
            </div>
        </template>

        <template v-if="value.type === 'single'">
            <span>抽到</span>
            <div class="cards-wrapper">
                <div class="cards">
                    <template v-for="card in value.cards">
                        <div class="card">
                            <select v-model="card.name" style="width: 160px">
                                <option v-for="name in cardNames" :value="name">{{name}}</option>
                            </select>
                            <button title="删除" @click="removeCard(card)" v-if="value.cards.length > 1">×</button>
                        </div>
                    </template>
                </div>
                <button title="添加卡片" @click="addCard">+</button>
            </div>
            <span>的数量</span>
            <select v-model="value.symbol">
                <option value="gt">&gt;</option>
                <option value="eq">=</option>
                <option value="lt">&lt;</option>
                <option value="neq">≠</option>
            </select>
            <input type="text" placeholder="数量" v-model="value.num">
            <div class="fill"></div>
            <button title="删除" v-if="!root" @click="remove">×</button>
        </template>
    </div>
</template>

<script>
    export default {
        name: 'condition-builder',
        props: {
            value: {},
            cardNames: {},
            root: Boolean
        },
        methods: {
            addGroup() {
                this.value.children = this.value.children || [];
                this.value.children.push({type: 'and', children: []});
            },
            addSingle() {
                this.value.children = this.value.children || [];
                this.value.children.push({type: 'single', cards: [{name: ''}], symbol: 'gt', num: 0});
            },
            remove() {
                this.$emit('remove', this.value);
            },
            onRemoveChild(child) {
                this.value.children = this.value.children.filter(curr => curr !== child);
            },
            addCard() {
                this.value.cards.push({name: ''})
            },
            removeCard(card) {
                this.value.cards = this.value.cards.filter(curr => curr !== card);
            }
        }
    };
</script>

<style lang="scss" scoped>
    .condition {
        user-select: none;
        min-width: 460px;

        button {
            cursor: pointer;
        }

        select {
            padding: 4px 8px;
        }

        &.single {
            display: flex;
            align-items: center;
            padding: 8px;
            background: #f2f2f2;
            border: solid 1px #aaa;

            & > *:not(:last-child) {
                margin: 0 .5em 0 0;
            }

            input {
                width: 4em;
                text-align: center;
            }

            select, input {
                height: 28px;
                box-sizing: border-box;
            }

            button {
                width: 28px;
                height: 28px;
            }

            .fill {
                flex: 1 1;
            }

            .cards-wrapper {
                display: flex;
                align-items: flex-end;

                .cards {
                    display: flex;
                    flex-direction: column;
                    margin-right: .5em;
                    align-items: flex-end;

                    .card {
                        display: flex;
                        align-items: center;

                        button {
                            width: 20px;
                            padding: 0;
                            margin-left: -1px;
                        }

                        &:not(:last-child) {
                            margin-bottom: 4px;
                        }

                        &:not(:first-child) {
                            &:before {
                                content: '+';
                                margin-right: 4px;
                            }
                        }
                    }
                }
            }
        }

        &.and, &.or {
            display: flex;
            flex-direction: column;
            box-sizing: border-box;
            border: solid 1px #aaa;

            .title {
                display: flex;
                align-items: center;
                padding: 8px;
                background: #f2f2f2;
                border-bottom: solid 1px #aaa;

                .fill {
                    flex: 1 1;
                }

                select {
                    height: 28px;
                    margin: 0 .5em;
                }

                button {
                    width: 28px;
                    height: 28px;
                }
            }

            .children {
                display: flex;
                flex-direction: column;
                flex: 1 1;
                min-height: 0;
                padding: 8px;
                overflow: auto;

                .buttons {
                    display: flex;
                    justify-content: flex-end;

                    a {
                        color: #409eff;
                        text-decoration: none;
                        line-height: 24px;

                        &:active {
                            color: #3a8ee6 !important;
                        }

                        &:not(:last-child) {
                            margin-right: .5em;
                        }
                    }
                }

                & > * {
                    &:not(:last-child) {
                        margin-bottom: 8px;
                    }
                }
            }
        }
    }
</style>