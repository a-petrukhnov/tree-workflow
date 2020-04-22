<template lang='pug'>
.action-editor(:class='{"is-visible": editorAvailable}')
    .action-editor__window
        .action-editor__content(v-if='editorAvailable')
            .action-editor__main(v-if='Object.keys(actionData.params).length')
                template(v-for='(item, index) in actionData.params')
                    label {{ index }}
                    input(
                        v-model='actionData.params[index]'
                        :name='index'
                    )
            .action-editor__buttons
                button(
                    :disabled='deleteDisabled'
                    type='button'
                    @click='deleteItem'
                ) Delete
                button(
                    type='button'
                    @click='cancelEdition'
                ) Cancel
                button(
                    type='button'
                    @click='saveItem'
                ) Save
    .action-editor__mask
</template>

<script>
export default {
    props: {
        action: {
            type: Object,
            required: false,
            default: () => ({})
        }
    },

    data() {
        return {
            actionData: {}
        }
    },

    computed: {
        editorAvailable() {
            return Object.keys(this.actionData).length;
        },

        deleteDisabled() {
            let hasChild = null;
            const conditions = this.action.conditions;

            if (conditions) {
                for (let i = 0; i < conditions.length; i++) {
                    if (conditions[i].actions && conditions[i].actions.length) {
                        hasChild = true;
                        break;
                    }
                }
            }

            return hasChild;
        }
    },

    mounted() {},

    watch: {
        action() {
            this.actionData = JSON.parse(JSON.stringify(this.action));
        }
    },

    methods: {
        deleteItem() {
            this.$emit('item-deleted', this.action.id);
        },

        cancelEdition() {
            this.$emit('edition-canceled');
        },

        saveItem() {
            this.$emit('item-saved', this.actionData);
            this.actionData = {};
        }
    }
}
</script>

<style lang='less'>
.action-editor {
    &__window {
        transition: transform 0.3s;
        left: 0;
        top: 0;
        position: fixed;
        transform: translateX(-100%);
        width: 400px;
        height: 100%;
        background: #fff;
        z-index: 10;
    }

    &__mask {
        transition: opacity 0.3s;
        left: 0;
        top: 0;
        position: fixed;
        z-index: -1;
        width: 100%;
        height: 100%;
        background: fade(#000, 40%);
        opacity: 0;
    }

    &__content {
        height: 100%;
        width: 400px;
        display: flex;
        flex-direction: column;
        overflow: hidden;
    }

    &__buttons {
        display: flex;
        flex-shrink: 0;
        padding: 16px;
        border-top: 1px solid #CFD8DC;
    }

    &__main {
        flex-grow: 1;
        padding: 16px;
        overflow: auto;
    }

    &.is-visible {
        .action-editor__window {
            transform: translateX(0);
        }

        .action-editor__mask {
            opacity: 1;
            z-index: 8;
        }
    }
}
</style>
