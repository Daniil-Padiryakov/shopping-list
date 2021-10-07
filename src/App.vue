<template>

    <div class="container max-w-screen-lg mx-auto px-4 bg-base-200 shadow-lg rounded-box">

        <div class="">
            <div class="mb-4">
                <h1 class="text-center text-4xl font-medium">Shopping List</h1>
            </div>

            <div class="tabs tabs-boxed flex justify-center mb-5">
                <a class="tab tab-lg"
                   :class="{'tab-active': currentTab === tab}"
                   v-for="tab in tabs" :key="tab"
                   @click="currentTab = tab">
                    {{tab}}
                </a>
            </div>
        </div>

        <component :is="currentTab" :state="state" :currentDate="currentDate"></component>



    </div>

</template>

<script>
    import Purchases from './components/Purchases.vue'
    import Statistics from './components/Statistics.vue'
    import Categories from './components/Categories.vue'

    export default {
        components: {
            Purchases, Statistics, Categories
        },
        data() {
            return {
                state: {
                    purchases: [],
                    categories: [],
                    nextPurchaseId: 0,
                },

                currentDate: {
                    year: null,
                    month: null,
                    day: null,
                },

                currentTab: 'Purchases',
                tabs: ['Purchases', 'Statistics', 'Categories']
            }
        },
        methods: {
            // вызывается при запуске приложения, берет данные из LC если есть и устанавливает текущую дату
            initApp() {
                this.setCurrentDate()
                if (this.getDataFromLC('stateOfShoppingList') !== null) {
                    this.state = this.getDataFromLC('stateOfShoppingList')
                }
            },
            setCurrentDate() {
                let date = new Date()
                this.currentDate.year = date.getFullYear()
                this.currentDate.month = date.getMonth()
                this.currentDate.day = date.getDate()
            },
            getDataFromLC(key) {
                let result = localStorage.getItem(key)
                return JSON.parse(result)
            },
        },
        mounted() {
            this.initApp()
        },
    }
</script>

<style>

</style>