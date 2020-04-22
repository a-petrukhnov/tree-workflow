<template lang='pug'>
.tree-graph(
    :class='treeClass'
    id='tree-graph'
    @dragover='onDragOver'
    @drop='onDrop'
)

    template(v-for='(child, index) in actionsList')
        tree-item(
            :key='child.id'
            :item-data='child'
            @item-edited='onItemEdited'
            @item-added='onItemAdded'
        )

    svg-connections(:connections='connections')
</template>

<script>
import TreeItem from './TreeItem.vue';
import SvgConnections from './SvgConnections.vue';

export default {
    name: 'Tree',

    components: {
        TreeItem,
        SvgConnections
    },

    props: {
        actions: {
            type: Array,
            default: () => ([])
        },
        deletedItem: {
            type: [Number, String],
            required: false,
            default: ''
        },
        savedItem: {
            type: Object,
            required: false,
            default: () => ({})
        }
    },

    data() {
        return {
            treeClass: '',
            actionsList: [],
            needDelete: '',
            needSave: {}
        };
    },

    computed: {
        connections() {
            const result = [];
            const actions = this.actionsList;

            const addConnection = (arr, parent = null, next = null, index = null) => {
                if (!arr.length && parent && next) {
                    result.push({
                        from: parent,
                        to: next,
                        type: 'branch_empty',
                        index
                    });
                } else {
                    for (let i = 0; i < arr.length; i++) {
                        const action = arr[i];

                        if (i === 0 && parent) {
                            result.push({
                                from: parent,
                                to: action.id,
                                type: 'branch_start',
                                index
                            });
                        }

                        if (action.conditions && action.conditions.length) {
                            for (let j = 0; j < action.conditions.length; j++) {
                                addConnection(
                                    action.conditions[j].actions,
                                    action.id,
                                    arr.length - 1 > i ? arr[i + 1].id : next,
                                    action.conditions[j].index
                                );
                            }
                        } else if (action.type !== 'exit' && i === arr.length - 1 && next) {
                            result.push({
                                from: action.id,
                                to: next,
                                type: 'branch_end',
                                index
                            });
                        } else if (i < arr.length - 1) {
                            result.push({
                                from: action.id,
                                to: arr[i + 1].id
                            });
                        }
                    }
                }
            }

            addConnection(actions);

            return result;
        }
    },

    mounted() {
        this.actionsList = this.actions;

        window.addEventListener('dragend', this.onDragEnd);
    },

    beforeDestroy() {
        window.removeEventListener('dragend', this.onDragEnd);
    },

    watch: {
        actionsList: {
            handler() {
                this.$emit('tree-changed', this.actionsList);
            },
            deep: true
        },

        deletedItem() {
            this.needDelete = this.deletedItem;
        },

        savedItem() {
            this.needSave = this.savedItem;
        },

        needDelete() {
            if (this.needDelete || this.needDelete === 0) {
                this.removeItem(this.needDelete);
                this.needDelete = '';
                this.$emit('item-edited', {});
            }
        },

        needSave() {
            if (Object.keys(this.needSave).length) {
                this.saveItem(this.needSave);
                this.needSave = {};
            }
        }
    },

    methods: {
        onDragEnd() {
            this.treeClass = '';
        },

        onDrop() {
            this.treeClass = '';
        },

        onDragOver(e) {
            e.preventDefault();
            e.dataTransfer.dropEffect = 'move';
            this.treeClass = 'dragover';
        },

        removeItem(id) {
            const actions = [...this.actionsList];

            const removeItem = (arr) => {
                let goal;

                for (let i = 0; i < arr.length; i++) {
                    const action = arr[i];

                    if (action.id === id) {
                        goal = i;
                        break;
                    }
                    else if (action.conditions) {
                        for (let i = 0; i < action.conditions.length; i++) {
                            removeItem(action.conditions[i].actions);
                        }
                    }
                }

                if (typeof goal === 'number') {
                    arr.splice(goal, 1);
                }
            }

            removeItem(actions);

            this.actionsList = actions;
        },

        saveItem(item) {
            const actions = [...this.actionsList];
            const id = item.id;

            const saveItem = (arr) => {
                let goal;

                for (let i = 0; i < arr.length; i++) {
                    const action = arr[i];

                    if (action.id === id) {
                        goal = i;
                        break;
                    }
                    else if (action.conditions) {
                        for (let i = 0; i < action.conditions.length; i++) {
                            saveItem(action.conditions[i].actions);
                        }
                    }
                }

                if (typeof goal === 'number') {
                    arr.splice(goal, 1, item);
                }
            }

            saveItem(actions);

            this.actionsList = actions;
        },

        onItemAdded({item, after, to, index}) {
            const actions = [...this.actionsList];

            const addItem = (arr) => {
                let goal;

                for (let i = 0; i < arr.length; i++) {
                    const action = arr[i];

                    if (action.id === after) {
                        goal = i;
                        break;
                    }
                    else if (action.conditions) {
                        if (action.id === to) {
                            const query = action.conditions.find(item => item.index === index);
                            if (!after) {
                                query.actions.unshift(item);
                            } else {
                                query.actions.push(item);
                            }
                            break;
                        } else {
                            for (let i = 0; i < action.conditions.length; i++) {
                                addItem(action.conditions[i].actions);
                            }
                        }
                    }
                }

                if (typeof goal === 'number') {
                    arr.splice(goal + 1, 0, item);
                }
            }

            addItem(actions);

            this.actionsList = actions;
        },

        onItemEdited({item}) {
            this.$emit('item-edited', item);
        },
    }
}
</script>

<style lang='less'>
@border-color: #CFD8DC;

.tree-graph {
    background-color: #fefefe;
    background-image: radial-gradient(#eee 15%, transparent 16%), radial-gradient(#eee 15%, transparent 16%);
    background-size: 10px 10px;
    background-position: 0 0, 30px 30px;
    text-align: center;
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: center;
    height: 100%;
    width: 100%;
    flex-grow: 1;
    padding: 16px;
    position: relative;

    &__dropzone {
        transform: translate(-50%, -50%);
        height: 60px;
        width: 150px;
        display: none;
        align-items: center;
        justify-content: center;
        position: absolute;
        left: 50%;
        top: 100%;
        z-index: 2;

        &::before {
            transform: translate(-50%, -50%);
            border: 1px dashed #CFD8DC;
            background: fade(#ECEFF1, 80%);
            height: 24px;
            width: 150px;
            position: absolute;
            left: 50%;
            top: 50%;
            content: '';
        }

        .svg-icon {
            width: 16px;
            height: 16px;
            fill: #80CBC4;
            position: relative;
        }
    }

    &__conditional {
        position: relative;
        border: 1px dotted #CFD8DC;
        background: fade(#ECEFF1, 20%);
        box-shadow: 0 0 2px 2px #fff inset;
        border-radius: 3px;
        display: flex;
        width: auto;
        margin: 4px 0;
        padding: 0 0 20px;
    }

    &__column {
        min-width: 150px;
        min-height: 50px;
        margin: 0 8px;
        position: relative;
        padding: 58px 0 0;

        > .tree-graph__dropzone {
            top: 58px;
        }

        > .tree-graph-item-holder:first-child {
            padding-top: 16px;
        }
    }

    &.dragover {
        background-color: darken(#fefefe, 2%);

        .tree-graph__dropzone {
            display: flex;
        }
    }
}

.tree-graph-condition {
    position: absolute;
    left: 50%;
    top: 28px;
    z-index: 2;
    transform: translate(-50%, -50%);
    background: #CFD8DC;
    padding: 2px 8px;
    border-radius: 5px;
}

.tree-graph-item-holder {
    text-align: center;
    position: relative;
    padding: 20px 0;

    &.draggable-prev > .tree-graph__dropzone,
    &.draggable-current > .tree-graph__dropzone {
        display: none
    }
}

.tree-graph-item {
    background: #fff;
    border: 1px solid @border-color;
    border-radius: 3px;
    width: 150px;
    min-height: 32px;
    padding: 8px;
    margin: 0 auto;
    display: flex;
    text-align: left;
    align-items: center;
    justify-content: center;
    flex-direction: column;
    position: relative;
    cursor: move;
    font-size: 12px;

    &.trigger {
        background: #C8E6C9;
    }

    &.exit {
        background: #E1F5FE;
    }

    &__edit {
        width: 32px;
        height: 32px;
        background: transparent;
        border: 0;
        position: absolute;
        top: 0;
        right: 0;
        cursor: pointer;

        &:disabled {
            cursor: default;
            opacity: 0.4;
        }

        .svg-icon {
            width: 20px;
            height: 20px;
            fill: #4CAF50;
        }
    }

    &__action {
        width: 20px;
        height: 20px;
        margin-right: 8px;
    }

    &__row {
        display: flex;
        align-items: center;
        justify-content: flex-start;
        width: 100%;
    }
}
</style>
