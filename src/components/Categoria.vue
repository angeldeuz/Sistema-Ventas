<template>
    <v-layout aling-start>
                <v-flex>
                    <v-toolbar flat color="white">
                        <v-toolbar-title>Categorias</v-toolbar-title>
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
                                    <v-flex xs12 sm12 md12>
                                        <v-text-field v-model="descripcion" label="Descripcion"></v-text-field>
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
                        <v-dialog v-model="adModal" max-width="290">
                            <v-card>
                                <v-card-title class="headline" v-if="adAccion==1">Activar Item?</v-card-title>
                                <v-card-title class="headline" v-if="adAccion==2">Desactivar Item?</v-card-title>
                                <v-card-text>
                                    Estas a punto de 
                                    <span v-if="adAccion==1">Activar</span>
                                    <span v-if="adAccion==2">Desactivar</span>
                                    el item {{ adNombre }}
                                </v-card-text>
                                <v-card-actions>
                                    <v-spacer></v-spacer>
                                    <v-btn color="green darken-1" flat="flat" @click="activarDescativarCerrar">
                                        Cancelar
                                    </v-btn>
                                    <v-btn v-if="adAccion==1" color="orange darken-4" flat="flat" @click="activar">
                                        Activar
                                    </v-btn>
                                    <v-btn v-if="adAccion==2" color="orange darken-4" flat="flat" @click="desactivar">
                                        Desactivar
                                    </v-btn>
                                </v-card-actions>
                            </v-card>
                        </v-dialog>
                </v-toolbar>
            <v-data-table
                :headers="headers"
                :items="categorias"
                :search="search"
                class="elevation-1"
            >
                <template v-slot:items="props">
                    <td class="justify-center layout px-0">
                        <v-icon small class="mr-2" @click="editItem(props.item)">
                        edit
                        </v-icon>
                        <template v-if="props.item.condicion">
                            <v-icon small @click="activarDesactivarMostrar(2,props.item)">
                                block
                            </v-icon>
                        </template>
                        <template v-else>
                            <v-icon small @click="activarDesactivarMostrar(1,props.item)">
                                check
                            </v-icon>
                        </template>     
                    </td>
                    <td>{{ props.item.nombre }}</td>
                    <td>{{ props.item.descripcion }}</td>
                    <td>
                        <div v-if="props.item.condicion">
                            <span class="blue--text">Activo</span>
                        </div>
                        <div v-else>
                            <span class="red--text">Inactivo</span>
                        </div>
                    </td>
                
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
                categorias:[],
                dialog: false,
                headers: [
                    { text: 'Opciones', value: 'opciones', sortable: false },
                    { text: 'Nombre', value: 'nombre' },
                    { text: 'Descripcion', value: 'descripcion',sortable: false },
                    { text: 'Estado', value: 'condicion',sortable: false }
                ],
                search: '',
                editedIndex: -1,
                id: '',
                nombre:'',
                descripcion:'',
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
            return this.editedIndex === -1 ? 'Nueva categoria' : 'Actualizar categoria'
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
                axios.get('api/Categorias/Listar',configuracion).then(function(response){
                    //console.log(response);
                    me.categorias=response.data;
                }).catch(function(error){
                    //console.log(error);
                });
            },
                editItem (item) {
                    this.id=item.idcategoria;
                    this.nombre=item.nombre;
                    this.descripcion=item.descripcion;
                    this.editedIndex=1;
                    this.dialog = true
                },

                deleteItem (item) {
                    const index = this.desserts.indexOf(item)
                    confirm('Are you sure you want to delete this item?') && this.desserts.splice(index, 1)
                },

                close () {
                    this.dialog = false;
                    this.limpiar();
                },

                limpiar(){
                    this.id="";
                    this.nombre="";
                    this.descripcion="";
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
                        axios.put('api/Categorias/Actualizar',{
                            'idcategoria': me.id,
                            'nombre': me.nombre,
                            'descripcion': me.descripcion
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
                        axios.post('api/Categorias/Crear',{
                            'nombre': me.nombre,
                            'descripcion': me.descripcion
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
                        this.validaMensaje.push("El nombre debe tener mas de 3 caracteres y menos de 50 caracteres");
                    }
                    if (this.validaMensaje.length){
                        this.valida=1;
                    }
                    return this.valida;
                },

                activarDesactivarMostrar(accion,item){
                    this.adModal=1;
                    this.adNombre=item.nombre;
                    this.adId=item.idcategoria;
                    if(accion==1){
                        this.adAccion=1;
                    }
                    else if (accion ==2){
                       this.adAccion=2;
                    }
                    else{
                        this.adModal=0;
                    }
                },
                activarDescativarCerrar(){
                    this.adModal=0;
                },
                activar(){
                    let me=this;
                    let header={"Authorization" : "Bearer " + this.$store.state.token};
                    let configuracion= {headers : header};
                        axios.put('api/Categorias/Activar/'+this.adId,{},configuracion).then(function(response){
                            me.adModal=0;
                            me.adAccion=0;
                            me.adNombre="";
                            me.id="";
                            me.listar();
                        }).catch(function(error){
                            console.log(error);
                        });
                },
                desactivar(){
                    let me=this;
                    let header={"Authorization" : "Bearer " + this.$store.state.token};
                    let configuracion= {headers : header};
                        axios.put('api/Categorias/Desactivar/'+this.adId,{},configuracion).then(function(response){
                            me.adModal=0;
                            me.adAccion=0;
                            me.adNombre="";
                            me.id="";
                            me.listar();
                        }).catch(function(error){
                            console.log(error);
                        });
                }
        },
    }
</script>
