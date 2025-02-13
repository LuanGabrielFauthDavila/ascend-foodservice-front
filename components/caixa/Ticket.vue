<template>
    <div class="main-caixa-ticket">
        <v-card min-height="100%" class="grid-sale">
            <v-card-title style="grid-area: title;">Caixa</v-card-title>
            <v-card-text style="grid-area: input;">
                <v-form v-model="valid" ref="form" lazy-validation>
                    <v-row dense>
                        <v-col cols="6">
                            <SearchText ref="selectedItem" :loading="loadingSelect" :headers="headersItem" :items="products" :mainLabel="'Produtos'" :subLabel="'Produto'" />
                        </v-col>
                        <v-col cols="3">
                            <v-text-field dense filled label="Qtde." v-model="quantity" :rules="rules" id="quantityField"></v-text-field>
                        </v-col>
                        <v-col cols="2">
                            <v-btn fab color="primary" @click="validate">
                                <v-icon>mdi-plus</v-icon>
                            </v-btn>
                        </v-col>
                    </v-row>
                </v-form>
            </v-card-text>
            <v-card-text style="grid-area: table;">
                <v-data-table :headers="headers" :items="form.products" :items-per-page="-1" hide-default-footer class="elevation-1">
                    <template v-slot:item.actions="{ item }">
                        <v-icon small class="mr-2" @click="delItem(item)">mdi-delete</v-icon>
                    </template>
                </v-data-table>
            </v-card-text>
            <v-card-text style="grid-area: actions;" class="d-flex flex-column justify-start align-center">
                <v-btn fab small color="success" class="mb-4" @click="addSale">
                    <v-icon>mdi-check</v-icon>
                </v-btn>
                <v-btn v-if="sale != null"
                    fab
                    small
                    color="primary"
                    :href="host+'/print/sale/'+sale"
                    target="_blank">
                    <v-icon>mdi-printer</v-icon>
                </v-btn>
                <v-btn v-else
                    fab
                    small
                    color="primary">
                    <v-icon>mdi-printer</v-icon>
                </v-btn>
            </v-card-text>
            <v-card-actions style="grid-area: info;">
                <v-row dense class="d-flex align-center">
                    <v-col cols="2">
                        <v-text-field dense filled label="Valor Frete" type="number" v-model="form.delivery"></v-text-field>
                    </v-col>
                    <v-col cols="2">
                        <v-text-field dense filled disabled label="Total Venda" type="number" v-model="totalSale"></v-text-field>
                    </v-col>
                </v-row>
            </v-card-actions>
        </v-card>
    </div>
    
</template>

<script>
import SearchText from '../default/SearchText.vue'
export default {
    name: "Ticket",
    components: {
        SearchText
    },
    data () {
        return {
            totalSale: 0.0,
            sale: null,
            host: process.env.HOST_BACK,
            valid: false,
            products: [],
            product: null,
            quantity: null,
            loadingSelect: false,
            loadingTable: false,
            form: {
                value: 0.0,
                delivery: 0,
                products: []
            },
            headersItem: [
                { 
                    text: "ID",
                    align: "center",
                    justify: "center",
                    value: "id"
                },
                { text: "Produto", value: "name" },
                { text: "Preço", value: "price" },
                { text: "Ações", value: "actions" },
            ],
            headers: [
                { 
                    text: "ID",
                    align: "center",
                    justify: "center",
                    value: "id"
                },
                { text: "Produto", value: "name" },
                { text: "Preço", value: "price" },
                { text: "Qtde.", value: "quantity" },
                { text: "Ações", value: "actions" },
            ],
            rules: [
                v => !!v || "Obrigatório",
            ],
        }
    },
    mounted () {
        this.getProduct()
    },
    methods: {
        validate() {
            this.$refs.form.validate()
            setTimeout(() => {
                if (this.valid == true) {
                    this.addItem()
                    this.$refs.form.reset()
                }
            }, 0)
        },
        reset () {
            this.$refs.form.reset()
            this.form.products = []
            this.totalSale = 0.0
        },
        addItem () {
            let objItem = {
                id: null,
                name: null,
                price: null,
                quantity: null,
            }
            if (this.form.products.filter((i) => {return i.id == this.$refs.selectedItem.item.id}).length == 0) {
                let product = this.$refs.selectedItem.items.filter((i) => {return i.id == this.$refs.selectedItem.item.id})[0]
                objItem["id"] = product.id
                objItem["name"] = product.name
                objItem["price"] = product.price
                objItem["quantity"] = this.quantity
                this.form.products.push(objItem)

                this.totalSale += parseFloat(product.price)*parseFloat(this.quantity)
            }
        },
        delItem (item) {
            this.form.products = this.form.products.filter((i) => { return i.id != item.id })
            this.totalSale -= parseFloat(item.price)*parseFloat(item.quantity)
        },
        async addSale () {
            for (let i = 0; this.form.products.length > i; i++) {
                this.form.value = parseFloat(this.form.value) + parseFloat(this.form.products[i].price)*parseFloat(this.form.products[i].quantity)
            }
            const req = await fetch(process.env.HOST_BACK+"/sale/?"+ new URLSearchParams({
                company: this.$route.params.company,
            }), {
                method: "POST",
                body: JSON.stringify(this.form),
                headers: new Headers({
                    "Authorization": `Token ${localStorage.getItem('token')}`,
                    "Content-Type": "application/json"
                })
            })
            if (req.status == 201) {
                const res = await req.json()
                this.sale = res.sale
                this.form.value = 0
                this.form.total = 0
                this.form.delivery = 0
                this.reset()
            }
        },
        async getProduct () {
            this.loadingSelect = true
            const req = await fetch(process.env.HOST_BACK+'/product/?'+new URLSearchParams({
                company: this.$route.params.company,
                type: 2,
            }), {
                method: "GET",
                headers: new Headers({
                    "Authorization": `Token ${localStorage.getItem('token')}`,
                    "Content-Type": "application/json"
                })
            })
            if (req.status == 200) {
                const res = await req.json()
                this.products = res
                this.loadingSelect = false
            }
        },
    }
}
</script>

<style scoped>
.main-caixa-ticket{
    grid-area: caixa;

    padding: 1vh 0 0 1vh;
}
.grid-sale {
    display: grid;
    grid-template-columns: 85% 15%;
    grid-template-rows: 15% 15% 55% 15%;
    grid-template-areas: 'title title' 
                         'input actions'
                         'table actions'
                         'info none';
}
</style>