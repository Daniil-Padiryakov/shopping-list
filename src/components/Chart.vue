<template>
    <div class="p-3 shadow-lg bg-base-100 rounded-box mb-3">
        <canvas ref="canvas"></canvas>
    </div>
</template>

<script>
    import { Bar, mixins } from 'vue-chartjs'

    export default {
        extends: Bar,
        mixins: [mixins.reactiveProp],
        props: ['options'],
        data() {
            options: {
                scales: {
                    y: {
                        beginAtZero: true
                    }
                }
            }
        },
        mounted () {
            this.renderChart(this.chartData, this.options)
        },
    }
</script>

<style scoped>
</style>