<template>
  <div class="container">
        <br>
        <div class="row">

          <div class="col-md-4 order-md-2 mb-4">
            <h4 class="d-flex justify-content-between align-items-center mb-3">
              <span class="text-muted">Carrito</span>
              <span class="badge badge-secondary badge-pill">{{listCarrito.length}}</span>
            </h4>
            <ul class="list-group mb-3">
              <li class="list-group-item d-flex justify-content-between lh-condensed">
                <div>
                  <h6 class="my-0">Producto</h6>
                  <small class="text-muted"></small>
                </div>
                <span class="text-muted">{{listCarrito.lentgh}}</span>
              </li>
              <li v-for="(itemcart,i) in listCarrito" class="list-group-item d-flex justify-content-between lh-condensed">
                <div>
                  <h6 class="my-0">{{itemcart.nombre}}</h6>
                  <small class="text-muted">{{itemcart.categoria}}</small>
                </div>

                <div>
                  <div>
                    <i class="material-icons"  v-on:click="cambiarCantidad(i,false)">remove_circle_outline</i>
                    <span class="text-muted"> {{itemcart.cant}}</span>
                    <i class="material-icons"  v-on:click="cambiarCantidad(i, true)">add_circle_outline</i>
                  </div>
                  <span class="text-muted"> {{convertMoney(itemcart.cant * itemcart.precio)}}</span>
                </div>

                <i class="material-icons"  v-on:click="deleteItem(i)">delete</i>

              </li>

              <li class="list-group-item d-flex justify-content-between">
                <span>Total (PESOS)</span>
                <strong> {{onViewTotal()}}</strong>
              </li>

              <li class="list-group-item d-flex justify-content-between">

                <select class="form-control" id="exampleFormControlSelect1"  v-model="SelectMesa" >
                  <option value="1">Mesa 1</option>
                  <option value="2">Mesa 2</option>
                  <option value="3">Mesa 3</option>
                  <option value="4">Mesa 4</option>
                  <option value="5">Mesa 5</option>
                </select>

              </li>


            </ul>

            <button type="button" class="btn btn-success btn-lg btn-block" v-on:click="onSendOrder()">PROCESAR PEDIDO</button>


          </div>


      <div class="col-md-8 order-md-1">

        <h4 class="mb-3">Food and Drink Bar</h4>

        <button type="button" class="btn btn-light" v-on:click="selectCategoria=0">Todos (All)</button>
        <button type="button" class="btn btn-light" v-for="cat in listCat" v-on:click="selectCategoria=cat.cat_id">{{cat.cat_nombre}}</button>

        <hr>

        <div class="row">

          <div class="card col-md-4 " v-show="prod.prod_categoria==selectCategoria||selectCategoria==0" style="padding:0px;" v-for="prod in listProd">
            <img class="card-img-top"  :src="prod.prod_image" alt="Card image cap">
            <div class="card-body">
              <h5 class="card-title">{{prod.prod_name}}</h5>
              <p class="card-text">{{prod.prod_description}}</p>
              <a href="#" class="btn btn-primary" v-on:click="addCart(prod)">{{convertMoney(prod.prod_price)}}</a>
            </div>
          </div>

        </div>


      </div>


    </div>


  </div> <!-- /container -->
</template>

<script>
    export default {
        data(){
          return{
            listCat:[],
            listProd:[],
            listCarrito:[],
            selectCategoria: 0,
            SelectMesa:1
          }
        },
        mounted() {
            this.listCatService()
            this.listProdService()
        },
        methods:{
          convertMoney(value){

          const formatterPeso = new Intl.NumberFormat('es-CO', {
             style: 'currency',
             currency: 'COP',
             minimumFractionDigits: 0
           })
           let valueFinal = formatterPeso.format(value);


           return valueFinal

          },
          listProdService(){

            axios.get("api/Producto/visible")
            .then(response=>{
              // cargar datos
              this.listProd = response.data
            })
            .catch(error=>{
              alert(error)
            })

          },
          listCatService(){

            axios.get("api/Categoria/disponible")
            .then(response=>{
              // cargar datos
              this.listCat = response.data
            })
            .catch(error=>{
              alert(error)
            })

          },
          addCart(item){
            //this.listCarrito.push(item)

            const itemcar = {
              id : item.prod_id,
              nombre : item.prod_name,
              categoria: item.cat_nombre,
              cant: 1,
              precio: item.prod_price
            }

            this.listCarrito.push(itemcar)
            //alert(JSON.stringify(item))
          },
          deleteItem(i){
            this.listCarrito.splice(i,1)
          },
          cambiarCantidad(i,type){

              // sacar variable de carrito
              const dataCar = this.listCarrito

              // sacar la cantidad de producto
              let cantd = dataCar[i].cant;

              if (type) {
                cantd = cantd + 1
              }
              else if (type==false&&cantd>=1) {
                cantd = cantd - 1
              }

              if ((type==false&&cantd>=1)||type) {
                dataCar[i].cant = cantd
                this.listCarrito
              }


          },
          onViewTotal(){
            let total = 0
            this.listCarrito.map((data)=>{
              total = total + (data.cant * data.precio)
            })

            return this.convertMoney(total)
          },
          onSendOrder()
          {

            // si es mayor e igual de 1 producto
            if (this.listCarrito.length>=1) {

              // sumar todal de los productos
              let total = 0
              this.listCarrito.map((data)=>{
                total = total + (data.cant * data.precio)
              })

              const formData = new FormData()
              // array de carrito se convierte en string con el JSON.stringify
              formData.append('pedidos',JSON.stringify(this.listCarrito))
              // select de mesa
              formData.append('mesa',this.SelectMesa)
              // total de carrito
              formData.append('total',total)

              axios.post("api/Orden/create",formData)
              .then(response => {

                //alert("Enviado exitosamente");

                this.$swal.fire(
                  'Enviado exitosamente!',
                  'Se envio los orden?? exitosamente',
                  'success'
                )

                // limpia el carrito
                this.listCarrito = []
              })
              .catch(error => {
                alert(error)
              })

            }


          }


        }

    }
</script>
