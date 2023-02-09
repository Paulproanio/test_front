<template>
<div>
    <v-card-text></v-card-text>
    <v-card width="700" class="mx-auto">
        <v-card-title class="primary white--text">SEVUELOS</v-card-title>
        <v-card-text></v-card-text>

        <v-card-text >
            <v-layout wrap>
                <v-flex md2>
                    <v-btn class="primary" @click="gestionarVuelo()">
                        <v-icon>add</v-icon>
                    </v-btn>
                </v-flex>
                <v-flex md2>
                    <v-btn class="primary" @click="traerVuelos()">
                        <v-icon>list</v-icon>
                    </v-btn>
                </v-flex>
                <v-flex md4>
                    <v-text-field v-model="buscar" @change="mostrarResultados(buscar)" append-icon="search" outlined dense label="Destination.." class="ml-3 mr-3"></v-text-field>
                </v-flex>
                <v-flex md4>
                    <v-text-field v-model="identificado" @change="mostrarResultadosXId(identificado)" append-icon="search" outlined dense label="ID.." type="number" class="ml-3 mr-3"></v-text-field>
                </v-flex>

            </v-layout>
        </v-card-text>
        <v-card-text v-if="idMostrar > 0">
            <v-layout wrap>

                <v-flex md3>
                    <v-chip label color="primary"> {{ idMostrar }}</v-chip>

                </v-flex>
                <v-flex md3>
                    <v-chip label color="primary"> {{ passengerMostrar }}</v-chip>

                </v-flex>
                <v-flex md3>
                    <v-chip label color="primary"> {{ destinationMostrar }}</v-chip>

                </v-flex>
                <v-flex md3>
                    <v-chip label color="primary"> {{ statusMostrar }}</v-chip>

                </v-flex>
            </v-layout>
        </v-card-text>

        <v-card-text>
            <v-data-table :items="elementos" :headers="cabeceras" dense>
                <template v-slot:item="{ item }">
                    <tr>
                        <td>{{ item.id }}</td>
                        <td>{{ item.passenger }}</td>
                        <td>{{ item.destination }}</td>
                        <td>
                            <div v-if="item.status == 'NEW'">
                                <v-chip label class="primary" small> {{ item.status }}</v-chip>
                            </div>
                            <div v-if="item.status == 'RESERVED'">
                                <v-chip label class="success" small> {{ item.status }}</v-chip>
                            </div>

                        </td>
                        <td>
                            <v-tooltip bottom>
                                <template v-slot:activator="{ on, attrs }">
                                    <v-btn icon @click="editarVuelo(item)">
                                        <v-icon color="primary" v-bind="attrs" v-on="on">edit</v-icon>
                                    </v-btn>
                                </template>
                                <span>Editar</span>
                            </v-tooltip>
                        </td>
                    </tr>
                </template>
            </v-data-table>
        </v-card-text>
    </v-card>
    <v-dialog v-model="dialogoVuelos" width="600" persistent>
        <v-card width="600">
            <v-card-title class="primary white--text">{{ tituloFormulario }} </v-card-title>
            <v-card-text></v-card-text>
            <v-card-text>
                <v-layout wrap>
                    <v-flex md12>
                        <v-text-field outlined dense v-model="editarElementos.passenger" label="Passenger.."></v-text-field>
                    </v-flex>
                    <v-flex md12>
                        <v-text-field outlined dense v-model="editarElementos.destination" label="Destination.."></v-text-field>
                    </v-flex>
                    <v-flex md6>
                        <v-text-field outlined dense v-model="editarElementos.status" label="Destination.." disabled></v-text-field>
                    </v-flex>
                </v-layout>
            </v-card-text>
            <v-card-actions>
                <v-btn class="error" @click="encerarDatos()">
                    <v-icon>close</v-icon>
                </v-btn>
                <v-btn class="primary" @click="guardarVuelo(editarElementos.passenger, editarElementos.destination)">
                    <v-icon>save</v-icon>
                </v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>
</div>
</template>

<script>
import axios from 'axios'
export default {
    name: 'SevueloFrontendVueHomeView',

    data() {
        return {
            idMostrar: '',
            passengerMostrar: '',
            destinationMostrar: '',
            statusMostrar: '',
            buscar: '',
            identificado: '',
            dialogoVuelos: false,
            passenger: '',
            destination: '',
            indiceEditado: -1,
            editarElementos: [{
                id: '',
                passenger: '',
                destination: '',
                status: ''
            }],
            defaultElementos: [{
                id: '',
                passenger: '',
                destination: '',
                status: ''
            }],
            elementos: [{
                id: '',
                passenger: '',
                destination: '',
                status: ''
            }],
            cabeceras: [{
                    text: 'Id',
                    value: 'id'
                },
                {
                    text: 'Passenger',
                    value: 'passenger'
                },
                {
                    text: 'Destination',
                    value: 'destination'
                },
                {
                    text: 'Status',
                    value: 'status'
                },
                {
                    text: 'Acciones',
                    value: ''
                }
            ]
        };
    },

    mounted() {
        this.traerVuelos()
    },

    methods: {

        mostrarResultadosXId(id) {
            this.elementos = []
            axios.get('http://localhost:8081/api/requests/' + id)
                .then(res => {

                    this.idMostrar = res.data.id;
                    this.passengerMostrar = res.data.passenger;
                    this.destinationMostrar = res.data.destination;
                    this.statusMostrar = res.data.status;

                })
                .catch(err => {
                    console.error(err);
                })
        },

        mostrarResultados(buscar) {
            this.elementos = []
            axios.get('http://localhost:8081/api/requests/bxname/' + buscar)
                .then(res => {
                    this.elementos = res.data;
                })
                .catch(err => {
                    console.error(err);
                })
        },

        encerarDatos() {
            this.dialogoVuelos = false
            this.$nextTick(() => {
                this.editarElementos = Object.assign({}, this.defaultElementos)
                this.indiceEditado = -1
            })
        },

        guardarVuelo(pass, dest, sta) {

            if (this.indiceEditado > -1) {
                let vueloEdit = {
                    id: this.indiceEditado,
                    passenger: pass,
                    destination: dest,
                    status: 'RESERVED'
                }

                axios.put('http://localhost:8081/api/reserve', vueloEdit)
                    .then(res => {
                        this.dialogoVuelos = false;
                        this.traerVuelos()
                        this.encerarDatos()

                    })
                    .catch(err => {
                        console.error(err);
                    })

            } else {
                let vueloNew = {
                    id: '',
                    passenger: pass,
                    destination: dest,
                    status: sta
                }
                axios.post('http://localhost:8081/api/requests', vueloNew)
                    .then(res => {
                        this.dialogoVuelos = false;
                        this.traerVuelos()
                        this.encerarDatos()
                    })
                    .catch(err => {
                        console.error(err);
                    })

            }

        },

        editarVuelo(item) {
            this.indiceEditado = item.id;
            this.editarElementos = Object.assign({}, item)
            this.dialogoVuelos = true

        },

        editarVuelo(item) {
            this.indiceEditado = item.id;
            this.editarElementos = Object.assign({}, item)
            this.dialogoVuelos = true

        },

        gestionarVuelo() {
            this.dialogoVuelos = true;
        },

        traerVuelos() {
            axios.get('http://localhost:8081/api/requests/')
                .then(res => {
                    this.elementos = res.data;

                })
                .catch(err => {
                    console.error(err);
                })
        }

    },

    computed: {
        tituloFormulario() {
            return this.indiceEditado === -1 ? 'NUEVO VUELO' : 'RESERVAR VUELO'
        }
    }
};
</script>
