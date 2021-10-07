<template>
    <div>
        <div class="flex justify-center mb-3 p-3 shadow-lg bg-base-100 rounded-box">

            <label class="label" for="date-begin">
                <span class="label-text text-lg font-medium">Begin date</span>
            </label>
            <input class="input input-bordered" v-model="statisticsBeginDate" type="date" id="date-begin">

            <label class="label" for="date-end">
                <span class="label-text text-lg font-medium">End date</span>
            </label>
            <input class="input input-bordered" v-model="statisticsEndDate" type="date" id="date-end">

            <button @click="getStatisticsPurchasesHandler" class="btn btn-success self-center ml-3">Show
            </button>
        </div>

        <div class="p-3 shadow-lg bg-base-100 rounded-box flex-grow mb-4">
            <div class="overflow-x-auto mb-3">
                <table class="table w-full table-compact">
                    <thead>
                    <tr>
                        <th>Title</th>
                        <th>Category</th>
                        <th>Price</th>
                    </tr>
                    </thead>

                    <tbody>
                    <tr v-for="item in statisticsPurchases">
                        <td>{{item.title}}</td>
                        <td>{{item.category}}</td>
                        <td>{{item.cost}}</td>
                    </tr>
                    </tbody>
                </table>
            </div>
        </div>

        <AllCostOfTable :items="statisticsPurchases"></AllCostOfTable>

        <div class="tabs flex justify-center mb-3">
            <button @click="setDataDays()" class="btn btn-accent">Days</button>
            <button @click="setDataWeeks()" class="btn btn-accent mx-5">Weeks</button>
            <button @click="setDataMonths()" class="btn btn-accent">Months</button>
        </div>

        <Chart :chart-data="datacollection"></Chart>

    </div>
</template>

<script>
    import Chart from './Chart.vue'
    import AllCostOfTable from './AllCostOfTable.vue'

    export default {
        components: {
            Chart, AllCostOfTable
        },
        props: {
            state: {
                type: Object,
            },
            currentDate: {
                type: Object,
            },
        },
        data() {
            return {
                statisticsBeginDate: null,
                statisticsEndDate: null,
                objsDateOfStatistics: [],
                statisticsPurchases: [],

                datacollection: {},
            }
        },
        methods: {
            // обработчик на кнопке, желательно отрефакторить
            getStatisticsPurchasesHandler() {
                this.objsDateOfStatistics = []
                for (let elem of this.getObjsOfRangeDate()) {
                    let obj = {
                        year: Number(elem.split('-')[0]),
                        month: this.removeZeroFromDate(elem.split('-')[1]) - 1,
                        day: Number(this.removeZeroFromDate(elem.split('-')[2])),
                    }
                    this.objsDateOfStatistics.push(obj)
                }
                this.statisticsPurchases = this.getStatisticsPurchases()
            },
            // получение массив объектов Date в промежутке дат
            getRangeOfDate(start, end) {
                let arr = []
                for (let dt = new Date(start); dt <= end; dt.setDate(dt.getDate() + 1)) {
                    arr.push(new Date(dt));
                }
                return arr;
            },
            // массив покупок для статистики
            getStatisticsPurchases() {
                this.statisticsPurchases = null
                let result = []
                for (let date of this.objsDateOfStatistics) {
                    for (let elem of this.state.purchases) {
                        if (JSON.stringify(elem.date) == JSON.stringify(date)) {
                            result.push(elem)
                        }
                    }
                }
                return result
            },
            getObjsOfRangeDate() {
                let objsOfRangeDate = this.getRangeOfDate(new Date(this.statisticsBeginDate), new Date(this.statisticsEndDate))
                objsOfRangeDate = objsOfRangeDate.map((v) => v.toISOString().slice(0, 10))
                return objsOfRangeDate
            },
            removeZeroFromDate(num) {
                let arr = num.split('')
                if (arr[0] == 0) {
                    arr.splice(0, 1)
                }
                return arr.join('')
            },


            // графики
            // график по дням в выбранном месяце
            setDataDays() {
                this.datacollection = {
                    labels: this.range(1, this.getLastDayOfMonth(this.currentDate.year, this.currentDate.month)),
                    datasets: [
                        {
                            label: 'Data One',
                            backgroundColor: '#f87979',
                            data: this.getCostOfMonth().map(item => item.summ),
                        },
                    ]
                }
            },
            getCostOfMonth() {
                let result = this.rangeOfObjs(1, this.getLastDayOfMonth(this.currentDate.year, this.currentDate.month))
                let arr = this.state.purchases.filter(item => item.date.year == this.currentDate.year && item.date.month == this.currentDate.month)
                for (let elem of arr) {
                    for (let key in result) {
                        if (result[key].index == elem.date.day) {
                            result[key].summ += Number(elem.cost)
                        }
                    }
                }
                return result
            },
            range(begin, end) {
                let arr = []
                for (let i = begin; i <= end; i++) {
                    arr.push(i)
                }
                return arr
            },
            rangeOfObjs(begin, end) {
                let arr = []
                for (let i = begin; i <= end; i++) {
                    arr.push({
                        index: i,
                        summ: null,
                    })
                }
                return arr
            },
            getLastDayOfMonth(year, month) {
                var date = new Date(year, month + 1, 0);
                return date.getDate();
            },

            // график недели, просто делит месяц по 7 дней, не по дням недели
            setDataWeeks() {
                this.datacollection = {
                    labels: this.range(1, this.chunk(this.getCostOfMonth(), 7).length),
                    datasets: [
                        {
                            label: 'Data One',
                            backgroundColor: '#f87979',
                            data: this.getWeek(),
                        },
                    ]
                }
            },
            getWeek() {
                let arr = this.chunk(this.getCostOfMonth(), 7)
                let result = []
                for (let subArr of arr) {
                    let sum = 0
                    for (let elem of subArr) {
                        sum += elem.summ
                    }
                    result.push(sum)
                }
                return result
            },
            chunk(arr, n) {
                let result = [];
                let count = Math.ceil(arr.length / n);
                for (let i = 0; i < count; i++) {
                    let elems = arr.splice(0, n);
                    result.push(elems);
                }
                return result;
            },

            // график месяцев в выбранном году
            setDataMonths() {
                this.datacollection = {
                    labels: this.range(1, 12),
                    datasets: [
                        {
                            label: 'Data One',
                            backgroundColor: '#f87979',
                            data: this.getMonths().map(item => item.summ),
                        },
                    ]
                }
            },

            getMonths() {
                let result = this.rangeOfObjs(0, 11)
                let arr = this.state.purchases.filter(item => item.date.year == this.currentDate.year)
                for (let elem of arr) {
                    for (let key in result) {
                        if (result[key].index == elem.date.month) {
                            result[key].summ += Number(elem.cost)
                        }
                    }
                }
                return result
            },
        },
        created() {
            this.setDataDays()
        }
    }
</script>

<style scoped>

</style>