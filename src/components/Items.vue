<template>
    <div class="items">
        <form class="add-item" @reset="disableSubmitButton(true)">
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
                    placeholder="Nombre (obligatorio)"
                    required
                    @keyup="dontAllowBlankFields()"
                />
                <input
                    class="form-control"
                    type="number"
                    ref="precio"
                    step="any"
                    name="precio"
                    placeholder="Precio (obligatorio)"
                    required
                    @keyup="dontAllowBlankFields()"
                />
                <input
                    class="form-control"
                    type="number"
                    ref="calidad"
                    name="calidad"
                    placeholder="Calidad (obligatorio) - max 50"
                    required
                    @keyup="dontAllowBlankFields()"
                />
                <input
                    class="form-control"
                    type="number"
                    ref="demanda"
                    name="demanda"
                    placeholder="Demanda (obligatorio) - max 100"
                    required
                    @keyup="dontAllowBlankFields()"
                />
                <input
                    class="form-control"
                    type="number"
                    ref="stock"
                    name="stock"
                    placeholder="Stock (obligatorio)"
                    required
                    @keyup="dontAllowBlankFields()"
                />
            </div>
            <input
                disabled
                type="submit"
                @click.prevent="addItem()"
                class="btn btn-primary"
                value="Añadir item"
                id="add-item"
            />
            <input
                type="reset"
                class="btn btn-danger"
                @click="dontAllowBlankFields()"
                value="Reset"
            />
        </form>
        <div class="mensajes">
            <span hidden class="mensaje" id="exito">Item creado</span>
            <span hidden class="mensaje" id="error"></span>
        </div>
        <button
            @click="inventarioItems()"
            id="mostrarInventarioBoton"
            type="button"
            class="btn btn-primary"
        >Mostrar inventario</button>
        <button
            @click="ocultarInventario()"
            id="ocultarInventarioBoton"
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
    mounted() {
        this.storedPacksVisibillity()

    },
    methods: {
        disableSubmitButton(disabled) {
            document.getElementById('add-item').disabled = disabled;
        },
        dontAllowBlankFields() {
            var areThereEmptyFields = false;
            let fields = document.querySelectorAll('input.form-control')
            for (var i = 0; i < fields.length; i++) {
                if (fields[i].required && !fields[i].value) {
                    areThereEmptyFields = true;
                }

            }
            this.disableSubmitButton(areThereEmptyFields);


        },


        storedPacksVisibillity() {
            if (localStorage.getItem("mostrarInventarioItems") == 'true') {
                this.inventarioItems();
                document.getElementById('mostrarInventarioBoton').hidden = true
                document.getElementById('ocultarInventarioBoton').hidden = false
            }
            else {
                document.getElementById('mostrarInventarioBoton').hidden = false
                document.getElementById('ocultarInventarioBoton').hidden = true
                this.ocultarInventario();
            }
        },
        // *******************       GET

        inventarioItems() {


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
                        this.changeInventButton(true);

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

        changeInventButton(mostrar) {
            document.getElementById('ocultarInventarioBoton').hidden = !mostrar
            document.getElementById('mostrarInventarioBoton').hidden = mostrar
            localStorage.setItem("mostrarInventarioItems", document.getElementById('mostrarInventarioBoton').hidden)

        },




        ocultarInventario() {
            document.getElementById("itemList").innerHTML = '';
            this.changeInventButton(false);

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
                        this.mensajeErrorOExito('exito')
                    } else {
                        console.log("Response Status:", response.status);
                        console.log("Reponse statuts text:", response.statusText);
                        this.mensajeErrorOExito('error', response.statusText)



                    }
                })
                .catch((error) => {
                    console.log(error.message);
                });
        },
        mensajeErrorOExito(status, error) {
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