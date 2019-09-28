<template>
    <v-layout align-start>
        <v-flex>
            <v-toolbar flat color="white">
                <v-toolbar-title>Ingresos</v-toolbar-title>
                    <v-divider
                    class="mx-2"
                    inset
                    vertical
                    ></v-divider>
                    <v-spacer></v-spacer>
                    <v-text-field v-if="verNuevo==0" class="text-xs-center" v-model="search" append-icon="search" label="Búsqueda" single-line hide-details></v-text-field>
                    <v-btn v-if="verNuevo==0" @click="listar" color="primary" dark class="mb-2">Buscar</v-btn>
                    <v-spacer></v-spacer>
                    <v-btn v-if="verNuevo==0" @click="mostrarNuevo" color="primary" dark class="mb-2">Nuevo</v-btn>
                    <v-dialog v-model="verArticulos" max-width="1000px">
                        <v-card>
                            <v-card-title>
                                <span class="headline">Selecciones un articulo</span>
                            </v-card-title>
                            <v-card-text>
                                <v-container grid-list-md>
                                    <v-layout wrap>
                                        <v-flex xs12 sm12 md12 lg12 xl12>
                                            <v-text-field append-icon="search" 
                                            class="text-xs-center" v-model="texto"
                                            label="Ingrese Articulo a buscar" @keyup.enter="listarArticulo">


                                            </v-text-field>
                                            <template>
                                                 <v-data-table
                                                    :headers="cabeceraArticulos"
                                                    :items="articulos"
                                                    class="elevation-1"
                                                >
                                                    <template slot="items" slot-scope="props">
                                                        <td class="justify-center layout px-0">
                                                            <v-icon
                                                            small
                                                            class="mr-2"
                                                            @click="agregarDetalle(props.item)"
                                                            >
                                                            add
                                                            </v-icon>
                                                        </td>
                                                        <td>{{ props.item.nombre }}</td>
                                                        <td>{{ props.item.categoria }}</td>
                                                        <td>{{ props.item.descripcion }}</td>
                                                        <td>{{ props.item.stock }}</td>
                                                        <td>{{ props.item.precio_venta }}</td>
                                                    </template>
                                                    <template slot="no-data">
                                                        <h3>No hay artículos para mostrar.</h3>
                                                    </template>
                                                </v-data-table>
                                            </template>
                                        </v-flex>
                                    </v-layout>
                                </v-container>
                            </v-card-text>
                            <v-card-actions>
                                <v-spacer>
                                    <v-btn @click="ocultarArticulos()" color="blue darken" flat>
                                        Cancelar
                                    </v-btn>
                                </v-spacer>
                            </v-card-actions>
                        </v-card>
                    </v-dialog>
                    <v-dialog v-model="adModal" max-width="290">
                        <v-card>
                            <v-card-title class="headline" v-if="adAccion==1">¿Activar Item?</v-card-title>
                            <v-card-title class="headline" v-if="adAccion==2">¿Anular Ingreso?</v-card-title>
                            <v-card-text>
                                Estás a punto de 
                                <span v-if="adAccion==1">Activar </span>
                                <span v-if="adAccion==2">Anular </span>
                                el ítem {{ adNombre }}
                            </v-card-text>
                            <v-card-actions>
                                <v-spacer></v-spacer>
                                <v-btn color="green darken-1" flat="flat" @click="activarDesactivarCerrar">
                                    Cancelar
                                </v-btn>
                                <v-btn v-if="adAccion==1" color="orange darken-4" flat="flat" @click="activar">
                                    Activar
                                </v-btn>
                                <v-btn v-if="adAccion==2" color="orange darken-4" flat="flat" @click="desactivar">
                                    Anular
                                </v-btn>
                            </v-card-actions>

                        </v-card>
                    </v-dialog>
                </v-toolbar>
            <v-data-table
                :headers="headers"
                :items="ingresos"
                :search="search"
                class="elevation-1"
                v-if="verNuevo==0"
            >
                <template slot="items" slot-scope="props">
                    <td class="justify-center layout px-0">
                        <v-icon small class="mr-2" @click="verDetalles(props.item)">
                        tab
                        </v-icon>
                        <template v-if="props.item.estado=='Aceptado'">
                            <v-icon small @click="activarDesactivarMostrar(2,props.item)">
                            block
                            </v-icon>
                        </template>
                    </td>
                    <td>{{ props.item.usuario }}</td>
                    <td>{{ props.item.proveedor}}</td>
                    <td>{{ props.item.tipo_comprobante }}</td>
                    <td>{{ props.item.serie_comprobante }}</td>
                    <td>{{ props.item.num_comprobante }}</td>
                    <td>{{ props.item.fecha_hora }}</td>
                    <td>{{ props.item.impuesto }}</td>
                    <td>{{ props.item.total }}</td>
                    <td>
                        <div v-if="props.item.estado=='Aceptado'">
                            <span class="blue--text">Aceptado</span>
                        </div>
                        <div v-else>
                            <span class="red--text">{{props.item.estado}}</span>
                        </div>
                    </td>
                </template>
                <template slot="no-data">
                <v-btn color="primary" @click="listar">Resetear</v-btn>
                </template>
            </v-data-table>
            <v-container grid-list-sm class="pa-4 white" v-if="verNuevo">
                <v-layout row wrap>
                    <v-flex xs12 sm4 md4 lg4 xl4>
                        <v-select v-model="tipo_comprobante" 
                        :items="comprobantes" label="Tipo Comprobante">
                        </v-select>
                    </v-flex>
                    <v-flex xs12 sm4 md4 lg4 xl4>
                        <v-text-field v-model="serie_comprobante" label="Serie Comprobante">
                        </v-text-field>
                    </v-flex>
                    <v-flex xs12 sm4 md4 lg4 xl4>
                        <v-text-field v-model="num_comprobante" label="Número Comprobante">
                        </v-text-field>
                    </v-flex>
                    <v-flex xs12 sm8 md8 lg8 xl8>
                        <v-select v-model="idproveedor"
                        :items="proveedores" label="Proveedor">
                        </v-select>
                    </v-flex>
                    <v-flex xs12 sm4 md4 lg4 xl4>
                        <v-text-field type="number" v-model="impuesto" label="Impuesto">
                        </v-text-field>
                    </v-flex>
                    <v-flex xs12 sm8 md8 lg8 xl8>
                        <v-text-field @keyup.enter="buscarCodigo()" v-model="codigo" label="Código">
                        </v-text-field>
                    </v-flex>
                    <v-flex xs12 sm2 md2 lg2 xl2>
                        <v-btn @click="mostrarArticulos()" small fab dark color="teal">
                            <v-icon dark>list</v-icon>
                        </v-btn>
                    </v-flex>
                    <v-flex xs12 sm2 md2 lg2 xl2 v-if="errorArticulo">
                        <div class="red--text" v-text="errorArticulo">
                        </div>
                    </v-flex>
                    <v-flex xs12 sm12 md12 lg12 xl12>
                        <v-data-table
                            :headers="cabeceraDetalles"
                            :items="detalles"
                            hide-actions
                            class="elevation-1"
                        >
                            <template slot="items" slot-scope="props">
                                <td class="justify-center layout px-0">
                                    <v-icon
                                    small
                                    class="mr-2"
                                    @click="eliminarDetalle(detalles,props.item)"
                                    >
                                    delete
                                    </v-icon>
                                </td>
                                <td>{{ props.item.articulo }}</td>
                                <td><v-text-field type="number" v-model="props.item.cantidad"></v-text-field></td>
                                <td><v-text-field type="number" v-model="props.item.precio"></v-text-field></td>
                                <td>$ {{ props.item.cantidad * props.item.precio}}</td>
                            </template>
                            <template slot="no-data">
                                <h3>No hay artículos agregados al detalle.</h3>
                            </template>
                        </v-data-table>
                        <v-flex class="text-xs-right">
                            <strong>Total Parcial: </strong>$ {{totalParcial=(total-totalImpuesto).toFixed(2)}}
                        </v-flex>
                        <v-flex class="text-xs-right">
                            <strong>Total Impuesto: </strong>$ {{totalImpuesto=((total*impuesto)/(100+impuesto)).toFixed(2)}}
                        </v-flex>
                        <v-flex class="text-xs-right">
                            <strong>Total Neto: </strong>$ {{total=(calcularTotal).toFixed(2)}}
                        </v-flex>
                    </v-flex>
                    <v-flex xs12 sm12 md12 lg12 xl12>
                        <div class="red--text" v-for="v in validaMensaje" :key="v" v-text="v">
                        </div>
                    </v-flex>
                    <v-flex xs12 sm12 md12 lg12 xl12>
                        <v-btn @click="ocultarNuevo" color="blue darken-1" flat>Cancelar</v-btn>
                        <v-btn v-if="verDet == 0" @click="guardar()" color="success">Guardar</v-btn>
                    </v-flex>
		        </v-layout>
            </v-container>
        </v-flex>
    </v-layout>
</template>
<script>
    import axios from 'axios'
    export default {
        data(){
            return {
                ingresos:[],                
                dialog: false,
                headers: [
                    { text: 'Opciones', value: 'opciones', sortable: false },
                    { text: 'Usuario', value: 'usuario' },
                    { text: 'Proveedor', value: 'proveedor' },
                    { text: 'Tipo Comprobante', value: 'tipo_comprobante' },
                    { text: 'Serie Comprobante', value: 'serie_comprobante', sortable: false  },
                    { text: 'Número Comprobante', value: 'num_comprobante', sortable: false  },
                    { text: 'Fecha', value: 'fecha_hora', sortable: false  },
                    { text: 'Impuesto', value: 'impuesto', sortable: false  },
                    { text: 'Total', value: 'total', sortable: false  },
                    { text: 'Estado', value: 'estado', sortable: false  }                
                ],
                cabeceraDetalles: [
                    { text: 'Borrar', value: 'borrar', sortable: false },
                    { text: 'Artículo', value: 'articulo', sortable: false },
                    { text: 'Cantidad', value: 'cantidad', sortable: false  },
                    { text: 'Precio', value: 'precio', sortable: false  },
                    { text: 'Subtotal', value: 'subtotal', sortable: false  }                
                ],
                detalles:[],
                search: '',
                id: '',
                idproveedor:'',
                proveedores:[                   
                ],
                tipo_comprobante: '',
                comprobantes: ['FACTURA','BOLETA','TICKET','GUIA'],
                serie_comprobante: '',
                num_comprobante: '',
                impuesto: 18,
                codigo:'',
                verNuevo:0,
                errorArticulo:null,
                totalParcial:0,
                totalImpuesto:0,
                total:0,
                cabeceraArticulos:[
                    {text:'Seleccionar', value:'seleccionar',sortable:false},
                    {text:'Articulo', value:'articulo'},
                    {text:'Categoria', value:'categoria'},
                    {text:'Descripcion', value:'descripcion',sortable:false},
                    {text:'Stock', value:'stock',sortable:false},
                    {text:'Precio Venta', value:'precio_venta',sortable:false}

                ],
                articulos:[],
                texto:'',
                verArticulos:0,
                verDet:0,
                valida: 0,
                validaMensaje:[],
                adModal: 0,
                adAccion: 0,
                adNombre: '',
                adId: ''             
            }
        },
        computed: {
            calcularTotal:function(){
                var resultado=0.0;
                for(var i=0;i<this.detalles.length;i++){
                    resultado=resultado+(this.detalles[i].precio*this.detalles[i].cantidad);
                }
                return resultado;
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
        methods:{
            mostrarNuevo(){
                this.verNuevo=1;
            },
            ocultarNuevo(){
                this.verNuevo=0;
                this.limpiar();
            },
            buscarCodigo(){
                let me=this;
                me.errorArticulo=null;
                let header={"Authorization" : "Bearer " + this.$store.state.token};
                let configuracion= {headers : header};
                axios.get('api/Articulos/BuscarCodigoIngreso/'+this.codigo,configuracion)
                .then(function(response){
                    //console.log(response);
                    me.agregarDetalle(response.data);
                }).catch(function(error){
                    console.log(error);
                    me.errorArticulo='No existe el artículo';
                });
            },
            listarArticulo(){
                let me=this;
                let header={"Authorization" : "Bearer " + this.$store.state.token};
                let configuracion= {headers : header};
                axios.get('api/Articulos/ListarIngreso/'+me.texto,configuracion).then(function(response){
                    //console.log(response);
                    me.articulos=response.data;
                }).catch(function(error){
                    console.log(error);
                });
            },
            mostrarArticulos(){
                this.verArticulos=1;
            },
            ocultarArticulos(){
                this.verArticulos=0;
            },
            agregarDetalle(data = []){
                this.errorArticulo=null;
                if (this.encuentra(data['idarticulo'])){
                    this.errorArticulo="El artículo ya ha sido agregado."
                }
                else{
                    this.detalles.push(
                        {idarticulo:data['idarticulo'],
                        articulo: data['nombre'],
                        cantidad:1,
                        precio:1}
                    );
                }
                
            },
            encuentra(id){
                var sw=0;
                for(var i=0;i<this.detalles.length;i++){
                    if (this.detalles[i].idarticulo==id){
                        sw=1;
                    }
                }
                return sw;
            },
            eliminarDetalle(arr,item){
                var i= arr.indexOf(item);
                if (i!==-1){
                    arr.splice(i,1);
                }
            },
            listar(){
                let me=this;
                let header={"Authorization" : "Bearer " + this.$store.state.token};
                let configuracion= {headers : header};
                let url='';
                if (!me.search){
                    url='api/Ingresos/Listar';
                }else{
                    url='api/Ingresos/ListarFiltro/'+me.search;
                }
                axios.get(url,configuracion).then(function(response){
                    //console.log(response);
                    me.ingresos=response.data;
                }).catch(function(error){
                    console.log(error);
                });
            },
            listarDetalles(id){
                let me=this;
                let header={"Authorization" : "Bearer " + this.$store.state.token};
                let configuracion= {headers : header};
                axios.get('api/Ingresos/ListarDetalles/'+id,configuracion).then(function(response){
                    //console.log(response);
                    me.detalles=response.data;
                }).catch(function(error){
                    console.log(error);
                });
            },
            verDetalles(item){
                this.limpiar();
                this.tipo_comprobante=item.tipo_comprobante,
                this.serie_comprobante=item.serie_comprobante,
                this.num_comprobante=item.num_comprobante,
                this.idproveedor=item.idproveedor,
                this.impuesto=item.impuesto,
                this.listarDetalles(item.idingreso);
                this.verNuevo=1;
                this.verDet=1;
            },
            select(){
                let me=this;
                var proveedoresArray=[];
                let header={"Authorization" : "Bearer " + this.$store.state.token};
                let configuracion= {headers : header};
                axios.get('api/Personas/SelectProveedores',configuracion).then(function(response){
                    proveedoresArray=response.data;
                    proveedoresArray.map(function(x){
                        me.proveedores.push({text: x.nombre,value:x.idpersona});
                    });
                }).catch(function(error){
                    console.log(error);
                });
            },
            limpiar(){
                this.id="";
                this.idproveedor="";
                this.tipo_comprobante="";
                this.serie_comprobante="";
                this.num_comprobante="";
                this.impuesto="18";
                this.codigo="";
                this.detalles=[];
                this.total=0;
                this.totalImpuesto=0;
                this.totalParcial=0;
                this.verDet=0;
            },
            guardar () {
                if (this.validar()){
                    return;
                }
                let header={"Authorization" : "Bearer " + this.$store.state.token};
                let configuracion= {headers : header};
                    //Código para guardar
                    let me=this;
                    axios.post('api/Ingresos/Crear',{
                        'idproveedor':me.idproveedor,
                        'idusuario':me.$store.state.usuario.idusuario,
                        'tipo_comprobante': me.tipo_comprobante,
                        'serie_comprobante':me.serie_comprobante,
                        'num_comprobante':me.num_comprobante,
                        'impuesto': me.impuesto,
                        'total':me.total,
                        'detalles':me.detalles
                    },configuracion).then(function(response){
                        me.ocultarNuevo();
                        me.listar();
                        me.limpiar();                        
                    }).catch(function(error){
                        console.log(error);
                    });
            },
            validar(){
                this.valida=0;
                this.validaMensaje=[];
        

                
                if (!this.idproveedor){
                    this.validaMensaje.push("Seleccione un proveedor.");
                }
                if (!this.tipo_comprobante){
                    this.validaMensaje.push("Seleccione un tipo de comprobante.");
                }
                if (!this.num_comprobante){
                    this.validaMensaje.push("Ingrese el numero de comprobante.");
                }
                if (!this.impuesto || this.impuesto<0){
                    this.validaMensaje.push("Ingrese un impuesto valido.");
                }
                if (this.detalles.length<=0){
                    this.validaMensaje.push("Ingrese al menos un articulo al detalle.");
                }
                if (this.validaMensaje.length){
                    this.valida=1;
                }
                return this.valida;
            },
            activarDesactivarMostrar(accion,item){
                this.adModal=1;
                this.adNombre=item.num_comprobante;
                this.adId=item.idingreso;                
                if (accion==1){
                    this.adAccion=1;
                }
                else if (accion==2){
                    this.adAccion=2;
                }
                else{
                    this.adModal=0;
                }
            },
            activarDesactivarCerrar(){
                this.adModal=0;
            },/*
            activar(){
                let me=this;
                let header={"Authorization" : "Bearer " + this.$store.state.token};
                let configuracion= {headers : header};
                axios.put('api/Usuarios/Activar/'+this.adId,{},configuracion).then(function(response){
                    me.adModal=0;
                    me.adAccion=0;
                    me.adNombre="";
                    me.adId="";
                    me.listar();                       
                }).catch(function(error){
                    console.log(error);
                });
            },
            */
            desactivar(){
                let me=this;
                let header={"Authorization" : "Bearer " + this.$store.state.token};
                let configuracion= {headers : header};
                axios.put('api/Ingresos/Anular/'+this.adId,{},configuracion).then(function(response){
                    me.adModal=0;
                    me.adAccion=0;
                    me.adNombre="";
                    me.adId="";
                    me.listar();                       
                }).catch(function(error){
                    console.log(error);
                });
            }
        }        
    }
</script>
