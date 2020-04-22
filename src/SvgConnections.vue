<template lang='pug'>
svg.svg-connections
    defs
        marker(
            id='arrow'
            markerWidth='5'
            markerHeight='5'
            refX='2.5'
            refY='5'
            viewBox='0 0 5 5'
        )
            polygon(
                points='0,0 5,0 2.5,5'
                fill='currentColor'
            )
    g
        template(v-for='connection in connectionsPaths')
            path(
                :d='connection'
                stroke='#B0BEC5'
                marker-end='url(#arrow)'
            )
</template>

<script>
export default {
    name: 'SvgConnections',

    props: {
        connections: {
            type: Array,
            default: () => ([])
        }
    },

    data() {
        return {
            treeLeft: 0,
            treeTop: 0,
            connectionsPaths: []
        }
    },

    mounted() {
        const treeEl = document.getElementById('tree-graph').getBoundingClientRect();

        this.treeLeft = treeEl.left; 
        this.treeTop = treeEl.top;

        window.addEventListener('resize', this.onResize);

        this.createConnectionsPaths();
    },

    beforeDestroy() {
        window.removeEventListener('resize', this.onResize);
    },

    watch: {
        connections() {
            this.onResize();
        }
    },

    methods: {
        onResize() {
            this.$nextTick(() => {
                this.createConnectionsPaths();
            });
        },

        getElementPoints(id, start, custom) {
            const el = document.querySelector(`[${custom || "data-connection-id"}="${id}"]`);
            const elRect = el && el.getBoundingClientRect();

            return {
                x: elRect ? (elRect.left - this.treeLeft + elRect.width / 2) : 0,
                y: elRect ? (elRect.top - this.treeTop + (start ? elRect.height : 0)) : 0
            };
        },

        createConnectionPath({from, to, type, index}) {
            const start = this.getElementPoints(from, true);
            const finish = this.getElementPoints(to);

            let result = '';

            if (type === 'branch_start' || type === 'branch_empty') {
                const mid = this.getElementPoints(`${from}_${index}`, false, 'data-condition-id');

                result += `M ${start.x},${start.y} L ${start.x},${mid.y - 12} `;
                result += `M ${start.x},${mid.y - 12} L ${mid.x},${mid.y - 12} `;

                if (type === 'branch_start') {
                    result += `M ${mid.x},${mid.y - 12} L ${finish.x},${finish.y}`;
                }
                if (type === 'branch_empty') {
                    result += `M ${mid.x},${mid.y - 12} L ${mid.x},${finish.y - 38} `;
                    result += `M ${mid.x},${finish.y - 38} L ${finish.x},${finish.y - 38} `;
                    result += `M ${finish.x},${finish.y - 38} L ${finish.x},${finish.y}`;
                }
            } else if (type === 'branch_end') {
                result += `M ${start.x},${start.y} L ${start.x},${finish.y - 38} `;
                result += `M ${start.x},${finish.y - 38} L ${finish.x},${finish.y - 38} `;
                result += `M ${finish.x},${finish.y - 38} L ${finish.x},${finish.y}`;
            } else {
                result += `M ${start.x},${start.y} L ${finish.x},${finish.y}`
            }

            return result;
        },

        createConnectionsPaths() {
            const result = [];

            for (const connection in this.connections) {
                result.push(this.createConnectionPath(this.connections[connection]));
            }

            this.connectionsPaths = result;
        }
    }
}
</script>

<style lang='less'>
.svg-connections {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
}
</style>
