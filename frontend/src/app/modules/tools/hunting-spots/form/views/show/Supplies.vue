<template>
    <panel>
        <div class="content">
            <h3>Supplies</h3>

            <div class="supplies">
                <div class="supply" v-for="supply in supplies">
                    <div class="amount">{{ supply.pivot.amount }}x</div>
                    <div class="thumb">
                        <img :src="image_path('item', supply.id)" />
                    </div>
                    <span class="title">{{ supply.title }}</span>
                </div>
            </div>

            <div v-if="equipments.length">
                <h3 class="margin-top-30">Equipments</h3>

                <div class="supplies">
                    <div class="supply" v-for="equipment in equipments">
                        <div class="thumb">
                            <img :src="image_path('item', equipment.id)" />
                        </div>
                        <span class="title">{{ equipment.title }}</span>
                    </div>
                </div>
            </div>
        </div>
    </panel>
</template>

<script>
    export default {
        props: ['supplies', 'equipments']
    }
</script>