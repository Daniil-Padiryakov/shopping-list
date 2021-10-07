<template>
    <div class="mb-3">

        <div class="flex mb-4">

            <div class="mr-3" style="max-width: 250px">
                <ul class="menu p-3 shadow-lg bg-base-100 rounded-box">
                    <li class="menu-title text-lg font-medium">Categories</li>
                    <li @click="setCategoriesPurchases(item)" v-for="item in state.categories"
                        class="">{{ item }}
                    </li>
                </ul>
            </div>

            <div class="p-3 shadow-lg bg-base-100 rounded-box flex-grow">
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
                        <tr v-for="item in categoriesPurchases">
                            <td>{{item.title}}</td>
                            <td>{{item.category}}</td>
                            <td>{{item.cost}}</td>
                        </tr>
                        </tbody>
                    </table>
                </div>
            </div>

        </div>

        <AllCostOfTable :items="categoriesPurchases"></AllCostOfTable>

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
        },
        data() {
            return {
                categoriesPurchases: [],
                datacollection: {},
            }
        },
        methods: {
            setCategoriesPurchases(category) {
                this.categoriesPurchases = this.getCategoriesPurchases(category)
            },
            getCategoriesPurchases(category) {
                let result = []
                for (let elem of this.state.purchases) {
                    if (elem.category == category) {
                        result.push(elem)
                    }
                }
                return result
            },
            setDataCategories() {
                this.datacollection = {
                    labels: this.state.categories,
                    datasets: [
                        {
                            label: 'Data One',
                            backgroundColor: '#f87979',
                            data: this.getAllCostOfEachCategory(),
                        },
                    ]
                }
            },
            getAllCostOfEachCategory() {
                let result = []
                for (let elem of this.state.categories) {
                    let purcOfCategory = this.getCategoriesPurchases(elem)
                    let allCost = null
                    for (let purc of purcOfCategory) {
                        allCost += Number(purc.cost)
                    }
                    result.push(allCost)
                }
                return result
            },
        },
        created() {
            this.categoriesPurchases = this.getCategoriesPurchases(this.state.categories[0])
            this.setDataCategories()
        }
    }
</script>

<style scoped>

</style>