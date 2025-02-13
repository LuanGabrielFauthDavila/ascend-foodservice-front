<template>
    <div>
        <v-card-title>Criar um novo Produto</v-card-title>
        <v-card-text>
            <v-form v-model="valid" ref="form" lazy-validation>
                <v-row dense>
                    <v-col cols="8">
                        <v-text-field dense filled label="Nome do Produto" v-model="form.name" :rules="rules"></v-text-field>
                    </v-col>
                    <v-col cols="3">
                        <v-select dense filled label="Marca" @focus="getBrand" v-model="form.brand" :rules="rules" :items="brands" item-text="brand" item-value="id"></v-select>
                    </v-col>
                    <v-col cols="1">
                        <AddBrand @getBrand="getBrand"></AddBrand>
                    </v-col>
                    <v-col cols="3">
                        <v-select dense filled label="Und. Medida" @focus="getMeasure" v-model="form.measure" :rules="rules" :items="measures" item-text="measure" item-value="id"></v-select>
                    </v-col>
                    <v-col cols="1">
                        <AddMeasure @getMeasure="getMeasure"></AddMeasure>
                    </v-col>
                    <v-col cols="3">
                        <v-text-field dense filled label="Qtd. Atual" v-model="form.stock" :rules="rules"></v-text-field>
                    </v-col>
                    <v-col cols="23">
                        <v-text-field dense filled label="Custo" v-model="form.cost" :rules="rules"></v-text-field>
                    </v-col>
                    <v-col v-if="!edit" cols="1" class="d-flex justify-center">
                        <v-btn fab color="success" @click="addProduct">
                            <v-icon>mdi-check</v-icon>
                        </v-btn>
                    </v-col>
                    <v-col v-else cols="1" class="d-flex justify-center">
                        <v-btn fab color="primary" @click="editProduct">
                            <v-icon>mdi-pencil</v-icon>
                        </v-btn>
                    </v-col>
                    <v-col cols="1" class="d-flex justify-center">
                        <v-btn fab color="warning" @click="reset">
                            <v-icon>mdi-close</v-icon>
                        </v-btn>
                    </v-col>
                </v-row>
            </v-form>
        </v-card-text>
    </div>
</template>

<script>
import AddBrand from './AddBrand.vue'
import AddMeasure from './AddMeasure.vue';
export default {
    name: "AddProduct",
    components: {
    AddBrand,
    AddMeasure
},
    emits: ["getProduct", "reset"],
    props: ['form', 'edit'],
    data() {
        return {
            dialog: false,
            valid: false,
            rules: [
                v => !!v || "Obrigatório",
            ],
            brands: [],
            measures: []
        };
    },
    mounted() {
        this.getBrand()
        this.getMeasure()
    },
    methods: {
        validate() {
            this.$refs.form.validate();
        },
        reset () {
            this.$emit('reset')
            this.$refs.form.reset();
        },
        addProduct() {
            this.validate();
            setTimeout(async () => {
                if (this.valid != false) {
                    this.form['company'] = this.$route.params.company
                    const req = await fetch(process.env.HOST_BACK + `/product/`, {
                        method: "POST",
                        body: JSON.stringify(this.form),
                        headers: new Headers({ 
                            "Authorization": `Token ${localStorage.getItem('token')}`,
                            "Content-Type": "application/json" 
                        }),
                    });
                    if (req.status == 201) {
                        this.dialog = false;
                        this.$emit("getProduct");
                        this.reset()
                    }
                    else {
                    }
                }
            }, 0);
        },
        async editProduct () {
            this.form['company'] = this.$route.params.company
            const req = await fetch(process.env.HOST_BACK + `/product/${this.form.id}/`, {
                method: "PATCH",
                body: JSON.stringify(this.form),
                headers: new Headers({ 
                    "Authorization": `Token ${localStorage.getItem('token')}`,
                    "Content-Type": "application/json" 
                }),
            })
            if (req.status == 201) {
                this.$emit("getProduct")
            }
        },
        async getBrand() {
            const req = await fetch(process.env.HOST_BACK + "/brand/?" + new URLSearchParams({
                company: this.$route.params.company,
            }), {
                method: "GET",
                headers: new Headers({
                    "Authorization": `Token ${localStorage.getItem('token')}`
                })
            })
            if (req.status == 200) {
                const res = await req.json();
                this.brands = res;
            }
        },
        async getMeasure() {
            const req = await fetch(process.env.HOST_BACK + "/measure/?" + new URLSearchParams({
                company: this.$route.params.company,
            }), {
                method: "GET",
                headers: new Headers({
                    "Authorization": `Token ${localStorage.getItem('token')}`
                })
            })
            if (req.status == 200) {
                const res = await req.json();
                this.measures = res;
            }
        }
    },
}
</script>
