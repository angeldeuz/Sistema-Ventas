<template>
    <v-layout aling-start>
                <v-flex>
                    <v-toolbar flat color="white">
                        <v-btn @click="crearPDF()"><v-icon>print</v-icon></v-btn>
                        <v-toolbar-title>Articulos</v-toolbar-title>
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
                                    <v-flex xs6 sm6 md6>
                                        <v-text-field v-model="codigo" label="Codigo"></v-text-field>
                                    </v-flex>
                                    <v-flex xs6 sm6 md6>
                                        <v-select v-model="idcategoria"
                                        :items="categorias" label="Categoria">
                                        </v-select>
                                    </v-flex>
                                    <v-flex xs12 sm12 md12>
                                        <v-text-field v-model="nombre" label="Nombre"></v-text-field>
                                    </v-flex>
                                     <v-flex xs6 sm6 md6>
                                        <v-text-field type="number" v-model="stock" label="Stock"></v-text-field>
                                    </v-flex>
                                     <v-flex xs6 sm6 md6>
                                        <v-text-field type="number" v-model="precio_venta" label="Precio Venta">

                                        </v-text-field>
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
                :items="articulos"
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
                        <td>{{ props.item.codigo }}</td>
                        <td>{{ props.item.nombre }}</td>
                        <td>{{ props.item.categoria }}</td>
                        <td>{{ props.item.stock }}</td>
                        <td>{{ props.item.precio_venta }}</td>
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
    import jsPDF from 'jspdf'
    import autoTable from 'jspdf-autotable';
    export default {
        data(){
            return{
                articulos:[],
                dialog: false,
                headers: [
                    { text: 'Opciones', value: 'opciones', sortable: false },
                    { text: 'Codigo', value: 'codigo', sortable: false },
                    { text: 'Nombre', value: 'nombre' },
                    { text: 'Categoria', value: 'categoria' },
                    { text: 'Stock', value: 'stock' },
                    { text: 'Precio Venta', value: 'precio_venta' },
                    { text: 'Descripcion', value: 'descripcion',sortable: false },
                    { text: 'Estado', value: 'condicion',sortable: false }
                ],
                search: '',
                editedIndex: -1,
                id: '',
                idcategoria:'',
                categorias:[],
                codigo:'',
                nombre:'',
                stock:0,
                precio_venta:0,
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
            return this.editedIndex === -1 ? 'Nuevo articulo' : 'Actualizar articulo'
            }
        },

        watch: {
            dialog (val) {
            val || this.close()
            }
        },

        created () {
            this.listar();
            this.select();
        },
        methods: {
            crearPDF(){
                var columns = [
                    {title:"Nombre", dataKey: "nombre"},
                    {title:"Codigo", dataKey: "codigo"},
                    {title:"Categoria", dataKey: "categoria"},
                    {title:"Stock", dataKey: "stock"},
                    {title:"Precio Venta", dataKey: "precio_venta"},

                ];
                var rows=[];
                this.articulos.map(function(x){
                    rows.push({nombre:x.nombre,codigo:x.codigo,categoria:x.categoria,stock:x.stock,precio_venta:x.precio_venta});
                });

                var doc = new jsPDF('p','pt');
                doc.autoTable(columns,rows,{
                    margin: {top: 60},
                    addPageContext:function(data){
                        doc.text("Listado de articulos", 40, 30);
                    }
                });
                doc.save('articulos.pdf');

            },
            listar(){
                let me=this;
                let header={"Authorization" : "Bearer " + this.$store.state.token};
                let configuracion= {headers : header};
                axios.get('api/Articulos/Listar',configuracion).then(function(response){
                    //console.log(response);
                    me.articulos=response.data;
                }).catch(function(error){
                    //console.log(error);
                });
            },
            select(){
                let me=this;
                let header={"Authorization" : "Bearer " + this.$store.state.token};
                let configuracion= {headers : header};
                var categoriasArray=[];
                axios.get('api/Categorias/Select',configuracion).then(function(response){
                    categoriasArray = response.data
                    categoriasArray.map(function(x){
                        me.categorias.push({text: x.nombre, value: x.idcategoria});
                    });
                }).catch(function(error){
                    //console.log(error);
                });
            },
                editItem (item) {
                    this.id=item.idarticulo;
                    this.idcategoria=item.idcategoria;
                    this.codigo=item.codigo;
                    this.nombre=item.nombre;
                    this.stock=item.stock;
                    this.precio_venta=item.precio_venta;
                    this.descripcion=item.descripcion;
                    this.editedIndex=1;
                    this.dialog = true
                },
                close () {
                    this.dialog = false;
                    this.limpiar();
                },

                limpiar(){
                    this.id="";
                    this.idcategoria="";
                    this.codigo="",
                    this.nombre="";
                    this.stock="";
                    this.precio_venta="";
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
                        axios.put('api/Articulos/Actualizar',{
                            'idarticulo': me.id,
                            'idcategoria': me.idcategoria,
                            'codigo': me.codigo,
                            'nombre': me.nombre,
                            'stock' : me.stock,
                            'precio_venta' : me.precio_venta,
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
                        axios.post('api/Articulos/Crear',{
                            'idcategoria': me.idcategoria,
                            'codigo': me.codigo,
                            'nombre': me.nombre,
                            'stock' : me.stock,
                            'precio_venta' : me.precio_venta,
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
                        this.validaMensaje.push("El nombre debe tener mas de 3 caracteres y menos de 50 caracteres.");
                    }
                    if(!this.idcategoria){
                        this.validaMensaje.push("Seleccione una Categotia.")
                    }
                    if(!this.stock || this.stock==0){
                        this.validaMensaje.push("Ingrese el stock inicial del articulo.")
                    }
                    if(!this.precio_venta || this.precio_venta==0){
                        this.validaMensaje.push("Ingrese el precio de venta del articulo.")
                    }
                    if (this.validaMensaje.length){
                        this.valida=1;
                    }
                    return this.valida;
                },

                activarDesactivarMostrar(accion,item){
                    this.adModal=1;
                    this.adNombre=item.nombre;
                    this.adId=item.idarticulo;
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
                        axios.put('api/Articulos/Activar/'+this.adId,{},configuracion).then(function(response){
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
                        axios.put('api/Articulos/Desactivar/'+this.adId,{},configuracion).then(function(response){
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
