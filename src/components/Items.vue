<template>
    <div class="items">
        <form class="add-item">
            <div class="inputsCaracteristicas">
                <div>
                    <label for="material">Elige material:</label>
                    <select ref="material" class="form-select" name="material" id="material">
                        <option value="normal">Normal</option>
                        <option value="indestructible">Indestructible</option>
                        <option value="consumible">Consumible</option>
                    </select>
                </div>
                <input
                    type="text"
                    class="form-control"
                    ref="nombre"
                    name="nombre"
                    placeholder="Nombre"
                    required
                />
                <input
                    class="form-control"
                    type="number"
                    ref="precio"
                    step="any"
                    name="precio"
                    placeholder="Precio"
                    required
                />
                <input
                    class="form-control"
                    type="number"
                    ref="calidad"
                    name="calidad"
                    placeholder="Calidad"
                    required
                />
                <input
                    class="form-control"
                    type="number"
                    ref="demanda"
                    name="demanda"
                    placeholder="Demanda"
                    required
                />
                <input
                    class="form-control"
                    type="number"
                    ref="stock"
                    name="stock"
                    placeholder="Stock"
                    required
                />
            </div>
            <input
                type="submit"
                @click.prevent="addItem()"
                class="btn btn-primary"
                value="Añadir item"
            />
            <input type="reset" class="btn btn-danger" value="Reset" />
        </form>
        <div class="mensajes">
            <span hidden class="mensaje" id="exito">Item creado</span>
            <span hidden class="mensaje" id="error"></span>
        </div>
        <button
            @click="inventario()"
            id="mostrarInventario"
            type="button"
            class="btn btn-primary"
        >Mostrar inventario</button>
        <button
            @click="ocultarInventario()"
            id="ocultarInventario"
            type="button"
            class="btn btn-danger"
            hidden
        >Ocultar inventario</button>
        <ul id="itemList" class="list-group">
            <li></li>
        </ul>
    </div>
</template>
<style>
.mensajes {
    padding-bottom: 15px;
}
.mensaje {
    font-size: large;
    font-weight: 500;
}
#exito {
    color: rgb(12, 192, 27);
}
#error {
    color: rgb(219, 50, 101);
}
.inputsCaracteristicas {
    display: flex;
    flex-direction: column;
    margin-left: 30%;
    margin-right: 30%;
}
.inputsCaracteristicas > input {
    margin: 4px;
}
form {
    margin-bottom: 30px;
}
</style>
<script>
//import $ from 'jquery'

export default {

    data() {
        return {
            serverip: '127.0.0.1:3000',
        }



    },
    created() {

    },
    methods: {




        // *******************       GET

        inventario() {


            var miHeaders = new Headers();

            var miInit = {
                method: 'GET',
                headers: miHeaders,
                mode: 'cors',
                // cambiarlo a force-cache => carga del disco
                cache: 'default'
            };


            fetch(`http://${this.serverip}/items`, miInit)
                .then((response) => {
                    if (response.ok) {
                        console.log("Response Status:", response.status);
                        console.log("Reponse statuts text:", response.statusText);
                        response.json().then((json) => {
                            this.logItems(json)
                            console.log(json)
                        })
                        this.changeInventButton();

                    } else {
                        console.log("Response Status:", response.status);
                        console.log("Reponse statuts text:", response.statusText);
                    }
                })
                .catch((error) => {
                    console.log(error.message);
                });
        },




        logItems(items) {
            const itemList = document.querySelector('#itemList');
            itemList.innerHTML = items.map((item, i) => {
                return `
                        <li  class="list-group-item">
                            <p id="item${i}"><span class="nombre item${i}"><b>${item.nombre}</b></span>
                                            precio:  ${item.precio}  
                                            calidad: ${item.calidad}  
                                            material:  ${item.material}
                                            demanda: ${item.demanda}  
                                            stock: ${item.stock}  


                                            </p>
                        </li>
                        `;
            }).join('');
        },
        //**************** BOTONES MOSTRAR/OCULTAR

        changeInventButton() {
            document.getElementById('ocultarInventario').hidden = !document.getElementById('ocultarInventario').hidden
            document.getElementById('mostrarInventario').hidden = !document.getElementById('ocultarInventario').hidden


        },




        ocultarInventario() {
            document.getElementById("itemList").innerHTML = '';
            this.changeInventButton();

        },

        //****************************                  POST



        addItem() {


            let data = {
                nombre: this.$refs.nombre.value,
                precio: this.$refs.precio.value,
                calidad: this.$refs.calidad.value,
                material: document.getElementById('material').value,
                demanda: this.$refs.demanda.value,
                stock: this.$refs.stock.value,



            };

            fetch(`http://${this.serverip}/items/add`, {
                method: 'POST',
                body: JSON.stringify(data),
                headers: {
                    'Content-Type': 'application/json'
                }
            })
                .then((response) => {
                    if (response.ok) {
                        console.log("Response OK Status:", response.status);
                        console.log("Reponse OK status text:", response.statusText);
                        this.messageOnErrorOrSuccess('exito')
                    } else {
                        console.log("Response Status:", response.status);
                        console.log("Reponse statuts text:", response.statusText);
                        this.messageOnErrorOrSuccess('error', response.statusText)



                    }
                })
                .catch((error) => {
                    console.log(error.message);
                });
        },
        mensajeErrorOExito(status, error) {
            {
                if (status === 'exito') {
                    document.getElementById('error').hidden = true;
                    document.getElementById('exito').hidden = false;
                }
                if (status === 'error') {
                    document.getElementById('exito').hidden = true;
                    document.getElementById('error').innerHTML = `Ha habido un error: ${error} `;
                    document.getElementById('error').hidden = false;
                }

            }
        }
    }





</script>