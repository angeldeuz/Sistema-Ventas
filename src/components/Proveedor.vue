<template>
    <v-layout aling-start>
                <v-flex>
                    <v-toolbar flat color="white">
                        <v-toolbar-title>Proveedores</v-toolbar-title>
                        <v-divider
                        class="mx-2"
                        inset
                        vertical
                        ></v-divider>
                        <v-spacer></v-spacer>
                        <v-text-field class="text-xs-center" v-model="search" append-icon="search" label="Busqueda" single-line hide-details></v-text-field>
                        <v-spacer></v-spacer>
                        <v-dialog v-model="dialog" max-width="500px">
                            <template v-slot:activator="{ on }">
                                <v-btn color="primary" dark class="mb-2" v-on="on">Nuevo</v-btn>
                            </template>
                            <v-card>
                                <v-card-title>
                                <span class="headline">{{ formTitle }}</span>
                                </v-card-title>
                    
                                <v-card-text>
                                <v-container grid-list-md>
                                    <v-layout wrap>
                                    <v-flex xs12 sm12 md12>
                                        <v-text-field v-model="nombre" label="Nombre"></v-text-field>
                                    </v-flex>
                                    <v-flex xs12 sm6 md6>
                                        <v-select v-model="tipo_documento"
                                        :items="documentos" label="Tipo Documento">
                                        </v-select>
                                    </v-flex>
                                     <v-flex xs12 sm6 md6>
                                        <v-text-field v-model="num_documento" label="Numero Documento"></v-text-field>
                                    </v-flex>
                                     <v-flex xs12 sm12 md12>
                                        <v-text-field v-model="direccion" label="Direccion">

                                        </v-text-field>
                                    </v-flex>
                                    <v-flex xs12 sm6 md6>
                                        <v-text-field v-model="telefono" label="Telefono">

                                        </v-text-field>
                                    </v-flex>
                                    <v-flex xs12 sm6 md6>
                                        <v-text-field v-model="email" label="email">

                                        </v-text-field>
                                    </v-flex>
                                    <v-flex xs12 sm12 md12 v-show="valida">
                                        <div class="red--text" v-for="v in validaMensaje" :key="v" v-text="v">

                                        </div>
                                    </v-flex>
                                    </v-layout>
                                </v-container>
                                </v-card-text>
                    
                                <v-card-actions>
                                    <v-spacer></v-spacer>
                                    <v-btn color="blue darken-1" flat @click="close">Cancelar</v-btn>
                                    <v-btn color="blue darken-1" flat @click="guardar">Guardar</v-btn>
                                </v-card-actions>
                            </v-card>
                        </v-dialog>
                </v-toolbar>
            <v-data-table
                :headers="headers"
                :items="proveedores"
                :search="search"
                class="elevation-1"
            >
                <template v-slot:items="props">
                    <td class="justify-center layout px-0">
                        <v-icon small class="mr-2" @click="editItem(props.item)">
                        edit
                        </v-icon>    
                    </td>
                        <td>{{ props.item.nombre }}</td>
                        <td>{{ props.item.tipo_persona }}</td>
                        <td>{{ props.item.tipo_documento }}</td>
                        <td>{{ props.item.num_documento }}</td>
                        <td>{{ props.item.direccion }}</td>
                        <td>{{ props.item.telefono }}</td>
                        <td>{{ props.item.email }}</td>
                </template>
                <template v-slot:no-data>
                <v-btn color="primary" @click="listar">Resetear</v-btn>
                </template>
            </v-data-table>
        </v-flex>
    </v-layout>
</template>

<script>
    import axios from 'axios'
    export default {
        data(){
            return{
                proveedores:[],
                dialog: false,
                headers: [
                    { text: 'Opciones', value: 'opciones', sortable: false },
                    { text: 'Nombre', value: 'nombre' },
                    { text: 'Tipo Persona', value: 'tipo_persona' },
                    { text: 'Tipo Documento', value: 'tipo_documento' },
                    { text: 'Numero Documento', value: 'num_documento' ,sortable: false},
                    { text: 'Direccion', value: 'direccion',sortable: false },
                    { text: 'Telefono', value: 'telefono',sortable: false },
                    { text: 'Email', value: 'email',sortable: false }
                ],
                search: '',
                editedIndex: -1,
                id: '',
                nombre:'',
                tipo_documento:'',
                documentos:['DNI','RUC','PASAPORTE','CEDULA'],
                num_documento:'',
                direccion:'',
                telefono:'',
                email:'',
                valida:0,
                validaMensaje:[],
                adModal:0,
                adAccion:0,
                adNombre:'',
                adId:''
            }
        },
        computed: {
            formTitle () {
            return this.editedIndex === -1 ? 'Nuevo proveedor' : 'Actualizar proveedor'
            }
        },

        watch: {
            dialog (val) {
            val || this.close()
            }
        },

        created () {
            this.listar();
        },
        methods: {

            listar(){
                let me=this;
                let header={"Authorization" : "Bearer " + this.$store.state.token};
                let configuracion= {headers : header};
                axios.get('api/Personas/ListarProveedores',configuracion).then(function(response){
                    //console.log(response);
                    me.proveedores=response.data;
                }).catch(function(error){
                    //console.log(error);
                });
            },
                editItem (item) {
                    this.id=item.idpersona;
                    this.nombre=item.nombre;
                    this.tipo_documento=item.tipo_documento;
                    this.num_documento=item.num_documento;
                    this.direccion=item.direccion;
                    this.telefono=item.telefono;
                    this.email=item.email;
                    this.editedIndex=1;
                    this.dialog = true
                },
                
                close () {
                    this.dialog = false;
                    this.limpiar();
                },

                limpiar(){
                    this.id="";
                    this.nombre="",
                    this.tipo_documento="";
                    this.num_documento="";
                    this.direccion="";
                    this.telefono="";
                    this.email="";
                    this.editedIndex=-1;
                },

                guardar () {
                    if(this.validar()){
                        return;
                    }
                    let header={"Authorization" : "Bearer " + this.$store.state.token};
                    let configuracion= {headers : header};
                    if (this.editedIndex > -1) {
                        //codigo para editar
                        let me=this;
                        axios.put('api/Personas/Actualizar',{
                            'idpersona': me.id,
                            'tipo_persona': 'Proveedor',
                            'nombre': me.nombre,
                            'tipo_documento':me.tipo_documento,
                            'num_documento': me.num_documento,
                            'direccion' : me.direccion,
                            'telefono' : me.telefono,
                            'email': me.email
                        },configuracion).then(function(response){
                            me.close();
                            me.listar();
                            me.limpiar();
                        }).catch(function(error){
                            console.log(error);
                        });
                    } else {
                        //codigo para guardar
                        let me=this;
                        axios.post('api/Personas/Crear',{
                            'tipo_persona': 'Proveedor',
                            'nombre': me.nombre,
                            'tipo_documento':me.tipo_documento,
                            'num_documento': me.num_documento,
                            'direccion' : me.direccion,
                            'telefono' : me.telefono,
                            'email': me.email
                        },configuracion).then(function(response){
                            me.close();
                            me.listar();
                            me.limpiar();
                        }).catch(function(error){
                            console.log(error);
                        });
                    }
                },

                validar(){
                    this.valida=0;
                    this.validaMensaje=[];
                    if (this.nombre.length<3 || this.nombre.length >50 ){
                        this.validaMensaje.push("El nombre debe tener mas de 3 caracteres y menos de 50 caracteres.");
                    }
                    if(!this.tipo_documento){
                        this.validaMensaje.push("Seleccione un tipo documento.")
                    }
                    if (this.validaMensaje.length){
                        this.valida=1;
                    }
                    return this.valida;
                }
        },
    }
</script>
