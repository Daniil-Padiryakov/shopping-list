<template>
    <div class="">

        <div class="flex justify-center mb-2 p-3 shadow-lg bg-base-100 rounded-box">

            <input class="input input-bordered" v-model="newTitle" type="text" placeholder="Title">

            <div class="category">
                <input class="input input-bordered" v-model="newCategory" type="text" placeholder="Category">

                <select class="select select-bordered max-w-full" v-model="newCategory">
                    <option v-for="category in state.categories" :value="category">
                        {{ category }}
                    </option>
                </select>
            </div>

            <input class="input input-bordered" v-model="newCost" type="text" placeholder="Cost">

        </div>

        <div class="flex justify-center mb-2">
            <button @click="addPurchase" type="button" class="btn btn-success">Add purchase</button>
        </div>


        <div class="flex justify-center mb-4 p-3 shadow-lg bg-base-100 rounded-box">

            <button @click="prevDay" type="button" class="btn btn-accent mr-3">Prev</button>

            <div class="stat-value">{{getCurrentDate}}</div>

            <button @click="nextDay" type="button" class="btn btn-accent ml-3">Next</button>

        </div>

    </div>

    <div class="overflow-x-auto mb-3">
        <table class="table w-full">
            <thead>
            <tr>
                <th>Title</th>
                <th>Category</th>
                <th>Price</th>
            </tr>
            </thead>

            <tbody>
            <tr v-for="item in getCurrentPurchases">
                <td>
                    <span v-on:dblclick="editPurchase(item)" v-if="!item.isEdited">{{item.title}}</span>
                    <input
                            class="input input-bordered"
                            @keypress.enter="editPurchaseDone(editPurchaseId)"
                            v-model="editPurchaseTitle"
                            v-if="item.isEdited"
                            type="text">
                </td>
                <td>{{item.category}}</td>
                <td>{{item.cost}}</td>
                <td>
                    <button @click="deletePurchase(item, $event)" class="btn btn-secondary btn-sm">Delete</button>
                </td>
            </tr>
            </tbody>
        </table>
    </div>

    <AllCostOfTable :items="getCurrentPurchases"></AllCostOfTable>

</template>

<script>
    import AllCostOfTable from './AllCostOfTable.vue'

    export default {
        components: {
            AllCostOfTable
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
                newTitle: '',
                newCategory: '',
                newCost: null,
                newDate: {
                    year: null,
                    month: null,
                    day: null,
                },

                editPurchaseId: null,
                editPurchaseTitle: '',
            }
        },
        methods: {
            addZeroToDate(num) {
                if (num >= 0 && num <= 9) {
                    return '0' + num;
                } else {
                    return num;
                }
            },

            getLastDay(year, month) {
                let date = new Date(year, month + 1, 0)
                return date.getDate()
            },

            //переключение на день назад
            prevDay() {
                this.currentDate.year = this.getPrevYear(this.currentDate.year, this.currentDate.month, this.currentDate.day)
                this.currentDate.day = this.getPrevDay(this.currentDate.day)
            },
            getPrevYear(year, month, day) {
                if (month == 0 && day == 1) {
                    return year - 1
                } else {
                    return year
                }
            },
            getPrevMonth(month) {
                if (month == 0) {
                    return 11
                } else {
                    return month - 1
                }
            },
            getPrevDay(day) {
                if (day == 1) {
                    this.currentDate.month = this.getPrevMonth(this.currentDate.month)
                    return this.getLastDayOfPrevMonth()
                } else {
                    return day - 1
                }
            },
            getLastDayOfPrevMonth() {
                return this.getLastDay(this.currentDate.year, this.currentDate.month)
            },
            getLastDayOfNextMonth() {
                return this.getLastDay(this.currentDate.year, this.currentDate.month)
            },

            //переключение на день вперед
            nextDay() {
                this.currentDate.year = this.getNextYear(this.currentDate.year, this.currentDate.month, this.currentDate.day)
                this.currentDate.day = this.getNextDay(this.currentDate.day)
            },
            getNextYear(year, month, day) {
                if (month == 11 && day == 31) {
                    return year + 1
                } else {
                    return year
                }
            },
            getNextMonth(month) {
                if (month == 11) {
                    return 0
                } else {
                    return month + 1
                }
            },
            getNextDay(day) {
                if (day == this.getLastDayOfNextMonth()) {
                    this.currentDate.month = this.getNextMonth(this.currentDate.month)
                    return 1
                } else {
                    return day + 1
                }
            },

            // добавление новой покупки
            addPurchase() {
                let newPurchase = {
                    id: this.state.nextPurchaseId++,
                    title: this.newTitle,
                    category: this.newCategory,
                    cost: this.newCost,
                    date: {
                        year: this.currentDate.year,
                        month: this.currentDate.month,
                        day: this.currentDate.day,
                    },
                    isEdited: false
                }
                this.state.purchases.push(newPurchase)
                this.addNewCategory()
                this.setDataToLC(this.state)
            },
            deletePurchase(purchase, event) {
                event.stopPropagation()
                let index = this.state.purchases.indexOf(purchase)
                this.state.purchases.splice(index, 1)
                this.setDataToLC(this.state)
            },
            editPurchase(purchase) {
                console.log(purchase)
                let index = this.state.purchases.indexOf(purchase)
                this.state.purchases[index].isEdited = true
                this.editPurchaseTitle = this.state.purchases[index].title
                this.editPurchaseId = index
            },
            editPurchaseDone(id) {
                this.state.purchases[id].title = this.editPurchaseTitle
                this.state.purchases[id].isEdited = false
                this.editPurchaseId = null
                this.editPurchaseTitle = ''
                this.setDataToLC(this.state)
            },
            addNewCategory() {
                if (!this.checkExistElemInArr(this.state.categories, this.newCategory) && this.newCategory != '') {
                    this.state.categories.push(this.newCategory)
                    this.setDataToLC(this.state)
                }
            },

            resetInputs() {
                this.newTitle = ''
                this.newCategory = ''
                this.newCost = null
            },
            checkExistElemInArr(arr, item) {
                let searchItem = arr.find(arrItem => arrItem === item)
                return searchItem !== undefined ? searchItem : undefined
            },
            setDataToLC(data) {
                localStorage.setItem('stateOfShoppingList', JSON.stringify(data))
            },
        },
        computed: {
            getCurrentPurchases() {
                return this.state.purchases.filter(item => JSON.stringify(item.date) === JSON.stringify(this.currentDate))
            },
            getCurrentDate() {
                return this.addZeroToDate(this.currentDate.day) + '.' +
                    this.addZeroToDate(this.currentDate.month + 1) + '.' +
                    this.currentDate.year
            },
        }
    }
</script>

<style>

</style>