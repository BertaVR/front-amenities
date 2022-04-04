<template>
    <div class="packs">
        <form class="add-item">
            <div id="inputsCaracteristicas">
                <input
                    class="form-control"
                    type="text"
                    ref="nombre"
                    name="nombre"
                    placeholder="Nombre"
                    required
                />
                <div id="itemList" ref="itemList">
                    <input
                        class="items form-control"
                        ref="item"
                        type="text"
                        placeholder="Item"
                        required
                    />
                </div>
            </div>
            <div class="inputs-botones">
                <div class="boton-superior">
                    <button @click="addItem" type="button" class="btn btn-success">
                        <svg
                            xmlns="http://www.w3.org/2000/svg"
                            width="16"
                            height="16"
                            fill="currentColor"
                            class="bi bi-plus"
                            viewBox="0 0 16 16"
                        >
                            <path
                                d="M8 4a.5.5 0 0 1 .5.5v3h3a.5.5 0 0 1 0 1h-3v3a.5.5 0 0 1-1 0v-3h-3a.5.5 0 0 1 0-1h3v-3A.5.5 0 0 1 8 4z"
                            />
                        </svg> Añadir Item
                    </button>
                </div>
                <div class="botones-inferiores">
                    <input
                        type="submit"
                        class="btn btn-primary add-pack"
                        @click="addPack"
                        value="Añadir Pack"
                    />
                    <input type="reset" class="btn btn-danger" value="Reset" />
                </div>
            </div>
        </form>
        <div class="mensajes">
            <span hidden class="mensaje" id="exito">Pack creado</span>
            <span hidden class="mensaje" id="error"></span>
        </div>
        <button
            id="mostrarInventarioBoton"
            @click="inventarioPacks"
            type="button"
            class="btn btn-primary"
        >Mostrar inventario</button>
        <button
            id="ocultarInventarioBoton"
            type="button"
            class="btn btn-danger"
            hidden
            @click="ocultarInventario"
        >Ocultar inventario</button>
        <ul id="packList" class="list-group">
            <li></li>
        </ul>
        <div
            class="modal fade bd-example-modal-lg"
            id="myModal"
            tabindex="-1"
            role="dialog"
            aria-labelledby="myLargeModalLabel"
            aria-hidden="true"
        ></div>
    </div>
</template>
<style>
ul {
    list-style: none;
}

.inputs-botones {
    display: flex;
    flex-direction: column;
    margin: 2px;
    padding: 2px;
}
.inputs-botones > div {
    margin: 5px;
}

.botones-inferiores > input {
    margin: 2px;
}

.items-incluidos {
    font-weight: bold;
    color: rgb(49, 0, 141);
}
#inputsCaracteristicas {
    display: flex;
    flex-direction: column;
    margin-left: 30%;
    margin-right: 30%;
}
#inputsCaracteristicas > div {
    display: flex;
    flex-direction: column;
}
#inputsCaracteristicas > div > input {
    margin: 4px;
}
#inputsCaracteristicas > input {
    margin: 4px;
}
form {
    margin-bottom: 30px;
}
button {
    margin-top: 4px;
}
</style>

<script>

//const $ = require('jquery')

import * as bootstrap from "bootstrap";
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
        storedPacksVisibillity() {
            if (localStorage.getItem("mostrarInventarioPacks") == 'true') {
                this.inventarioPacks();

                document.getElementById('mostrarInventarioBoton').hidden = true
                document.getElementById('ocultarInventarioBoton').hidden = false
            }
            else {
                document.getElementById('mostrarInventarioBoton').hidden = false
                document.getElementById('ocultarInventarioBoton').hidden = true
                this.ocultarInventario();
            }
        },
        inventarioPacks() {


            var miHeaders = new Headers();

            var miInit = {
                method: 'GET',
                headers: miHeaders,
                mode: 'cors',
                // cambiarlo a force-cache => carga del disco
                cache: 'default'
            };


            fetch(`http://${this.serverip}/packs`, miInit)
                .then((response) => {
                    if (response.ok) {
                        console.log("Response Status:", response.status);
                        console.log("Reponse statuts text:", response.statusText);
                        response.json().then((json) => {
                            this.logPacks(json);
                            console.log(json)
                        });
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




        logPacks(packs) {
            const packList = document.querySelector('#packList');
            packList.innerHTML = packs.map((pack, i) => {
                return `
                        <li  class="list-group-item">
                            <p id="pack${i}"><span class="nombre pack${i}"><b>${pack.nombre}</b></span>
                                            precio:  ${pack.precio}  
                                            stock:  ${pack.stock}
                                            calidad:  ${pack.calidad}  

                                            <ul>
                                        <lh class="items-incluidos"> Items incluidos:</lh>

                                            ${pack.items.map((item) => {
                    //TODO:  Refactor
                    return `
                          <li  class="item_in_pack">
                              <p id="item"><span class="nombre item"><b>${item.nombre}</b></span>
                                                                        material:  ${item.material}  
                                                                        calidad:  ${item.calidad}  
                                                                        precio:  ${item.precio}  
                                                                        demanda:  ${item.demanda}  

                                              </p>
                          </li>
                          `;
                }).join('')}
                                            </ul>
                                            </p>
                                         <button id="pack${i}" type="button" class="btn btn-danger borrarPack">Borrar pack</button>
                                         <button id="pack${i}" type="button" class="btn btn-warning editarPack">Ver pack</button>

                        </li>
                        `;
            }).join('');
            this.activarOpcionBorrar();
            this.activarOpcionEditar();
        },

        activarOpcionBorrar() { //refactor vue
            document.querySelectorAll('.borrarPack').forEach(b => {
                b.addEventListener('click', this.borrarPack);

            });
        },

        activarOpcionEditar() { //refactor vue
            document.querySelectorAll('.editarPack').forEach(b => {
                b.addEventListener('click', this.getPackToEdit);

            });
        },
        borrarPack(e) {
            let id_pack = e.target.id
            let nombre_pack = document.querySelector(`#${id_pack} span`).innerText
            if (confirm(`Borrar ${nombre_pack}`)) {

                fetch(`http://${this.serverip}/packs/${nombre_pack}`, {
                    method: 'DELETE',
                    headers: {
                        'Content-Type': 'application/json'
                    }
                })
                    .then((response) => {
                        if (response.ok) {
                            console.log("Response OK Status:", response.status);
                            console.log("Item borrado:", response.statusText);
                            this.inventarioPacks();
                        } else {
                            console.log("Response Status:", response.status);
                            console.log("Reponse statuts text:", response.statusText);
                        }
                    })
                    .catch((error) => {
                        console.log(error.message);
                    });
            }
        },

        changeInventButton(mostrar) {
            document.getElementById('ocultarInventarioBoton').hidden = !mostrar
            document.getElementById('mostrarInventarioBoton').hidden = mostrar
            localStorage.setItem("mostrarInventarioPacks", document.getElementById('mostrarInventarioBoton').hidden)

        },
        ocultarInventario() {
            document.getElementById("packList").innerHTML = '';
            this.changeInventButton(false);

        },

        addItem() {
            const addItemInput = document.createElement('input');
            addItemInput.placeholder = 'Item';
            addItemInput.type = 'text';
            const contenedor = document.getElementById('itemList');
            contenedor.appendChild(addItemInput);
            addItemInput.classList.add('items', 'form-control');

        },

        getItemsInCurrentPack() {
            let items = this.$refs.itemList.children
            let arrayItems = [];
            for (var i = 0; i < items.length; i++) {
                if (items[i].value)
                    arrayItems.push(items[i].value);
            }
            return arrayItems;
        },
        addPack(e) {
            e.preventDefault();

            let data = {
                nombre: this.$refs.nombre.value,
                items: this.getItemsInCurrentPack(),




            };

            fetch(`http://${this.serverip}/packs/add`, {
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
                document.getElementById('error').innerHTML = `Ha habido un error: ${error} <br> Recuerde añadir sólo items que existan actualmente en el inventario de items `;
                document.getElementById('error').hidden = false;
            }

        },
        getPackToEdit(e) {
            let id_pack = e.target.id
            let nombre_pack = document.querySelector(`#${id_pack} span`).innerText

            fetch(`http://${this.serverip}/packs/${nombre_pack}`, {
                method: 'GET',
                headers: {
                    'Content-Type': 'application/json'
                }
            })
                .then((response) => {
                    if (response.ok) {
                        console.log("Response OK Status:", response.status);
                        console.log("Item encontrado:", response.statusText);
                        response.json().then((json) => {
                            this.renderizarModalEditar(json);
                            console.log(json)
                        })
                    } else {
                        console.log("Response Status:", response.status);
                        console.log("Reponse statuts text:", response.statusText);
                    }
                })
                .catch((error) => {
                    console.log(error.message);
                });

        }
        ,
        renderizarModalEditar(pack) {
            document.getElementById('myModal').innerHTML = `
                    <div class="modal-dialog">
                        <div class="modal-content">
                            <div class="modal-header">
                                <h5 class="modal-title">Editar ${pack.nombre}</h5>
                                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                            </div>
                            <div class="modal-body">
                                <form>
                                <label for="nombre">Nombre del pack:</label><br>
                                     <input type="text" id="cambiarNombre" name="cambiarNombre" ref="cambiarNombre"value="${pack.nombre}""><br>
                                     ${pack.items.map((item, i) => {
                let numeroItem = i + 1
                //TODO:  Refactor
                return `
                         <label for="item${numeroItem}">Item ${numeroItem}:</label><br>
                                     <input type="text" class="item" name="item${numeroItem}" value="${item.nombre}"><br>
                          `;
            }).join('')}
                                </form>
                            </div>
                            <div class="modal-footer">
                                <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
                                <button type="button" id="cambiarNombre"  ">Cambiar Nombre</button>
                                                                         <!–– Desactivar botón editar ya que no funciona ––>

                            </div>
                        </div>
                    </div>`
            this.nombrePackActual = pack.nombre
            this.abrirModalEditar(pack)

        },
        abrirModalEditar() {
            var myModal = new bootstrap.Modal(document.getElementById('myModal'));
            myModal.show();
            this.activarOpcionCambiarNombre()

        },
        activarOpcionCambiarNombre() {
            document.querySelector('button#cambiarNombre').addEventListener('click', this.cambiarNombre);


        },

        cambiarNombre() {
            console.log('Cambiando nombre')

            fetch(`http://${this.serverip}/packs/${this.nombrePackActual}/cambiarNombre/${document.getElementById('cambiarNombre').value}`, {
                method: 'PUT',
                headers: {
                    'Content-Type': 'application/json'
                }
            })
                .then((response) => {
                    if (response.ok) {
                        console.log("Response OK Status:", response.status);
                        console.log("Reponse OK status text:", response.statusText);

                    } else {
                        console.log("Response Status:", response.status);
                        console.log("Reponse statuts text:", response.statusText);

                    }
                })
                .catch((error) => {
                    console.log(error.message);
                });

        }


    }
}








/* function */




/* TENGO UN PROBLEMA CON LAS PROMESAS
 
function renderItems(items) {
      items.map((item, i) => {
          return `
                      <li  class="item_in_pack">
                          <p id="item${i}"><span class="nombre item${i}"><b>${item.nombre}</b></span>
  
 
 
                                          </p>
                      </li>
                      `;
      }).join('');
 
  }*/










</script>