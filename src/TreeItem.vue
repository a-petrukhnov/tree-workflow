<template lang='pug'>
.tree-graph-item-holder
    .tree-graph-item(
        :class='itemData.type'
        :data-connection-id='itemData.id'
        draggable='true'
        @dragstart='(e) => onDragStart(e, itemData)'
        @dragend='onDragEnd'
    )
        button.tree-graph-item__edit(
            v-if='!(itemData.hasOwnProperty("edit") && itemData.edit === false)'

            @click='onItemEdited({item: itemData})'
        )
            edit.svg-icon

        .tree-graph-item__row
            component.tree-graph-item__action(:is='itemData.type')

            .tree-graph-item__title {{ (itemData.params && itemData.params.title) || itemData.type }}

    template(v-if='itemData.conditions')
        .tree-graph__conditional
                template(v-for='condition in itemData.conditions')
                    .tree-graph__column
                        .tree-graph-condition(
                            :data-condition-id='`${itemData.id}_${condition.index}`'
                        ) {{ String(condition.condition) }}

                        .tree-graph__dropzone(
                            @dragover='onDragOver'
                            @drop='(e) => onDrop(e, true, condition.index)'
                        )
                            add.svg-icon

                        template(v-if='condition.actions && condition.actions.length')
                            template(v-for='child in condition.actions')
                                tree-item(
                                    :key='child.id'
                                    :item-data='child'
                                    @item-edited='onItemEdited'
                                    @item-added='onItemAdded'
                                )

    .tree-graph__dropzone(
        v-if='itemData.type !== "exit"'
        @dragover='onDragOver'
        @drop='onDrop'
    )
        add.svg-icon
</template>

<script>
import TreeItem from './TreeItem.vue';

import Edit from './static/img/edit.svg';
import Add from './static/img/add.svg';

import Condition from './static/img/events/condition.svg';
import Email from './static/img/events/email.svg';
import Delay from './static/img/events/delay.svg';
import Exit from './static/img/events/exit.svg';
import NotificationAndroid from './static/img/events/notification-android.svg';
import NotificationWeb from './static/img/events/notification-web.svg';
import NotificationIos from './static/img/events/notification-ios.svg';
import Period from './static/img/events/period.svg';
import Sms from './static/img/events/sms.svg';
import Telegram from './static/img/events/telegram.svg';
import Trigger from './static/img/events/trigger.svg';
import Viber from './static/img/events/viber.svg';

export default {
    name: 'TreeItem',

    components: {
        TreeItem,
        Edit,
        Add,
        Condition,
        Email,
        Delay,
        Exit,
        NotificationAndroid,
        NotificationIos,
        NotificationWeb,
        Period,
        Sms,
        Telegram,
        Viber,
        Trigger
    },

    props: {
        itemData: {
            type: Object,
            required: true
        }
    },

    methods: {
        onDragOver(e) {
            e.preventDefault();
            e.dataTransfer.dropEffect = 'move';
        },

        onDrop(e, empty, index) {
            let item;

            try {
                item = JSON.parse(e.dataTransfer.getData('text/plain'));
                e.dataTransfer.clearData('text/plain');
            }
            catch(err) {
                console.log(err);
            }

            if (item && Object.keys(item).length) {
                if (item.replace) {
                    this.$emit('item-removed', {item});
                }

                this.$emit('item-added', {
                    item,
                    after: empty ? null : this.itemData.id,
                    to: empty ? this.itemData.id : null,
                    index
                });
            }
        },
        onDragEnd() {
            document.querySelector('.draggable-prev').classList.remove('draggable-prev');
            document.querySelector('.draggable-current').classList.remove('draggable-current');
        },

        onDragStart(e, data) {
            const parent = e.target.parentElement;
            const prev = e.target.parentElement.previousElementSibling;

            if (prev) {
                prev.classList.add('draggable-prev');
            }
            parent.classList.add('draggable-current');

            e.dataTransfer.setData('text/plain', JSON.stringify({
                ...data,
                replace: true
            }));
        },

        onItemEdited({item}) {
            this.$emit('item-edited', {item});
        },

        onItemAdded({item, after, to, index}) {
            this.$emit('item-added', {
                item,
                after,
                to,
                index
            });
        }
    }
}
</script>
