<template lang='pug'>
.components-library
    .components-library__list
        template(v-for='(item, index) in componentsList')
            .components-library__item(
                draggable='true'
                @dragstart='(e) => onDragStart(item, e)'
            )
                component.svg-icon(:is='item.type')
                div {{ item.type }}
</template>

<script>
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
import Viber from './static/img/events/viber.svg';

export default {
    components: {
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
        Viber
    },

    props: {
        componentsList: {
            type: Array,
            required: true
        }
    },

    methods: {
        onDragStart(data, e) {
            const result = Object.assign({}, data);
            result.id = `temp_${new Date().getTime()}`;
            e.dataTransfer.setData('text/plain', JSON.stringify(result));
        }
    }
};
</script>

<style lang='less'>

.components-library__item {
    background: #ECEFF1;
    border: 1px solid #CFD8DC;
    border-radius: 3px;
    min-width: 150px;
    min-height: 32px;
    padding: 8px;
    margin: 8px 0;
    display: flex;
    align-items: center;
    justify-content: flex-start;
    position: relative;
    cursor: move;
    font-size: 12px;

    .svg-icon {
        width: 20px;
        height: 20px;
        fill: #424242;
        margin-right: 8px;
    }
}
</style>
