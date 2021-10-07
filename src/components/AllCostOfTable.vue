<template>
    <div class="shadow stats mb-3">
        <div class="stat">
            <div class="stat-title">All cost:</div>
            <div class="stat-value text-primary">{{getCurrentPurchasesAllCost}}</div>
        </div>
    </div>
</template>

<script>
    export default {
        props: {
            items: {
                type: Array,
            },
        },
        computed: {
            getCurrentPurchasesAllCost() {
                let result = null
                if (this.items.length === 0) {
                    return 0
                } else {
                    for (let item of this.items) {
                        result += Number(item.cost)
                    }
                }
                return result
            },
        }
    }
</script>

<style scoped>

</style>