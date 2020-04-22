<template lang='pug'>
.scene
    library(:components-list='componentsList')
    tree(
        :actions='actions'
        :deleted-item='deletedItem'
        :saved-item='savedItem'
        @tree-changed='onTreeChanged'
        @item-edited='onItemEdited'
    )
    editor(
        :action='editedItem'
        @edition-canceled='editedItem = {}'
        @item-deleted='onItemDeleted'
        @item-saved='onItemSaved'
    )
</template>

<script>
import Tree from './Tree.vue';
import Library from './Library.vue';
import Editor from './Editor.vue';

export default {
    components: {
        Library,
        Tree,
        Editor
    },

    data() {
        return {
            actions: [
                {
                    id: new Date().getTime(),
                    type: 'trigger',
                    edit: false
                },
                {
                    id: new Date().getTime() + 1,
                    type: 'exit',
                    edit: false
                }
            ],

            componentsList: [
                {
                    id: 1,
                    type: 'sms',
                    params: {
                        title: ''
                    }
                },
                {
                    id: 2,
                    type: 'email',
                    params: {}
                },
                {
                    id: 3,
                    type: 'notification-web',
                    params: {}
                },
                {
                    id: 4,
                    type: 'notification-android',
                    params: {}
                },
                {
                    id: 5,
                    type: 'notification-ios',
                    params: {}
                },
                {
                    id: 6,
                    type: 'period',
                    params: {}
                },
                {
                    id: 7,
                    type: 'delay',
                    params: {}
                },
                {
                    id: 8,
                    type: 'viber',
                    params: {}
                },
                {
                    id: 9,
                    type: 'telegram',
                    params: {}
                },
                {
                    id: 10,
                    type: 'condition',
                    conditions: [
                        {
                            condition: true,
                            index: 0,
                            actions: []
                        },
                        {
                            condition: false,
                            index: 1,
                            actions: []
                        }
                    ]
                },
                {
                    id: 10,
                    type: 'exit'
                },
            ],

            editedItem: {},

            deletedItem: null,

            savedItem: {}
        }
    },

    methods: {
        onTreeChanged(payload) {
            this.actions = payload;
        },

        onItemEdited(payload) {
            this.editedItem = payload;
        },

        onItemDeleted(payload) {
            this.deletedItem = payload;
        },

        onItemSaved(payload) {
            this.savedItem = payload;
        }
    }
};
</script>

<style lang='less'>
* {
    box-sizing: border-box;
}

body,
html {
    height: 100%;
    margin: 0;
    font-family: 'Open Sans', sans-serif;
    font-size: 14px;
}

.scene {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    height: 100%;
    width: 100%;
    padding: 16px;

    .components-library {
        width: 200px;
    }

    .tree-graph {
        flex-grow: 1;
        margin-left: 16px;
    }
}
</style>
